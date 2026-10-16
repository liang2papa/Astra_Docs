# NPC 공격 반복·회전·추적 복귀 연결

작성: 2026-09-30. C++ 반영 후 사용자가 수행할 빌드·Editor 연결·PIE 검증 안내입니다.
빌드, UHT, Blueprint Compile, PIE는 아직 수행하지 않았습니다.

## 변경 내용

- 두뇌의 일회성 `NPC Set Attack Command` 대신 지속 Task `NPC Combat Attack`을 사용합니다.
- Task가 Running을 유지하며 NPC Controller의 공격 판단을 갱신합니다. GA를 직접 호출하지 않습니다.
- AI Focus를 현재 타겟으로 지정합니다. NPC 몸은 기존 `WeaponFacingRotationRateYaw` 속도로 회전합니다.
- `BP_NPCCharacter`의 `Attack Facing Tolerance Yaw` 이내로 몸 방향이 맞아야 공격 입력을 보냅니다. 기본값 5도는 튜닝용 초기값입니다.
- 다음 요청은 Upper의 `ReadyEntered`, 무기 Idle, GA 공격·재장전 태그 해제를 확인한 뒤 발신합니다.
- 요청이 Upper의 Fire/Reload 진입으로 이어지지 않았을 때만 `BP_NPCController`의 `Attack Request Retry Delay` 뒤 재시도합니다. 기본값 0.5초는 요청 복구용이며 발사 간격이 아닙니다.
- 탄약 부족 시 NPC 입력 해석이 Reload 입력으로 전환합니다. 실제 탄약 처리는 기존 Reload GA가 담당합니다.
- 몸 방향이 어긋난 진행 중 공격, 사거리 이탈, 타겟 상실, 사망, UnPossess, Controller EndPlay에서 공격을 정리합니다.
- Player C++과 Weapon Handler의 발사 정책은 변경하지 않았습니다.

## 1. 빌드

사용자가 Unreal Editor와 Visual Studio를 저장 후 종료하고 `Development Editor | Win64` 전체 빌드를 수행합니다.
새 StateTree USTRUCT와 UPROPERTY가 있으므로 Live Coding만으로 연결하지 않습니다.

## 2. ST_AIBrain의 Attack Tasks

기존 `NPC Set Attack Command / Start Attack=true`를 제거하고 `NPC Combat Attack`을 추가합니다.
순서는 다음과 같습니다.

1. `NPC Set Brain State`: Brain State = Attack
2. `NPC Set Chase Command`: Start Chase = false
3. `NPC Combat Attack`: AIController = 두뇌 Context의 AIController
4. Debug Text: 필요하면 기존 노드 유지

세 Task의 AIController를 모두 바인딩합니다. 출력의 `None`을 그대로 두지 않습니다.
`NPC Combat Attack`은 완료 판정에 포함하고, Attack의 Tasks Completion은 `All`로 설정합니다.
나머지 일회성 상태·명령 Task와 Debug Task는 기존처럼 완료 판정에서 제외합니다.
새 Task가 Running인 동안 Attack 상태는 유지됩니다. 완료 즉시 자신에게 재진입하는 루프는 추가하지 않습니다.
`On State Failed → Idle` 전이를 추가하여 Controller·Pawn·Mediator 연결 실패 시 복구합니다.
기존 Idle/Chase/Dead의 `NPC Set Attack Command / false`는 유지할 수 있습니다.

## 3. 사거리 이탈 → Chase 바인딩 복구

`Attack → Chase` 전이:

| 항목 | 값 |
|---|---|
| Trigger | On Tick |
| Target State | Chase |
| Condition 1 | NPC Has Current Target |
| Condition 1 / AIController | 두뇌 Context의 AIController |
| Condition 1 / Invert | false |
| Condition 2 | 타겟이 사정거리 안에 있는가 |
| Condition 2 / AIController | **두뇌 Context의 AIController에 반드시 바인딩** |
| Condition 2 / Range Override | 0: 장착 무기 EffectiveRange 사용 |
| Condition 2 / Invert | true |
| 두 Condition 관계 | AND |

첨부한 트리에서 Condition 2의 ID는 `4A6D41844CB3A10B90784197B8DF35F6`이며, 대응 바인딩이 없었습니다.
Controller가 없으면 Condition이 false를 반환하기 때문에 전이가 막힙니다.

`Attack → Idle`: On Tick, Has Current Target / Invert=true, AIController 동일 바인딩.
`Chase → Attack`: On Tick, 사거리 Condition / Invert=false, AIController 동일 바인딩.
`Chase → Idle`: On Tick, Has Current Target / Invert=true, AIController 동일 바인딩.
Root의 사망 전이는 기존대로 유지합니다.

## 4. NPC Upper 공격 Task를 한 번 활성화하도록 정리

현재 파일의 `ST_NPCUpperState`는 Player 경로의 `STT_Attack`을 참조합니다.
NPC Upper Attack 상태의 해당 Task를 `STT_NPC_Attack`으로 교체하고, 아래 흐름으로 NPC Task를 구성합니다.
Player의 `STT_Attack`은 이 작업에서 수정하지 않습니다.

`STT_NPC_Attack`의 부모에서 EntityCharacter와 EntityContext를 정상적으로 확보하고 StateTree 인스턴스에 바인딩합니다.
BeginPlay/외부 Player 조회를 사용하지 않습니다. GetContext는 부모 Context 반환 구현을 사용합니다.
Task에 bool `bAbilityStarted`를 추가합니다. 기본값 false입니다.

### EnterState 노드와 핀

1. `Event EnterState → Set bAbilityStarted=false`.
2. `Is Valid`로 EntityCharacter, EntityContext, Context.StateMediator, Context.StateEventBus, Context.AbilitySystem, Context.WeaponComp를 검사합니다.
   - 하나라도 Invalid: `Finish Task`, Succeeded=false.
3. `Publish Upper State`:
   - Target = EntityContext.StateEventBus
   - State Event = FireEntered
   - Source = EntityCharacter
4. `Try Activate Ability By Tag`:
   - Target = EntityContext.StateMediator
   - Ability Tag = Ability.Weapon.Fire
5. 실행 출력 → `Branch`, Condition = **Try Activate의 Return Value**.
   - true: `Set bAbilityStarted=true`, Task를 종료하지 않고 기다립니다.
   - false: `Finish Task`, Succeeded=false.

GA 활성화는 EnterState에서 한 번만 수행합니다. Tick의 `Try Activate Ability By Tag` 호출을 제거합니다.
Draw를 직접 실행하지 않습니다. 두뇌의 C++이 Draw만 요청하고 Ready와 방향 정렬을 확인한 뒤 공격 입력을 발신합니다.

### Tick 노드와 핀

1. `Event Tick → Branch(bAbilityStarted)`.
   - false: 대기.
   - true: Context.AbilitySystem과 Context.WeaponComp 유효성 검사. Invalid면 bAbilityStarted=false 후 Finish Task(false).
2. `Has Matching Gameplay Tag`:
   - Target = EntityContext.AbilitySystem
   - Tag to Check = State.Combat.Attacking
3. `Branch`, Condition = 태그 검사 Return Value.
   - true: GA 실행 중이므로 대기.
   - false: `Set bAbilityStarted=false → Finish Task`.
     - Succeeded 핀 = `EntityContext.WeaponComp → Did Last Fire Succeed`.

`GA_Attack`은 Activation Owned Tags에 `State.Combat.Attacking`을 갖고 정상·실패·취소 시 반드시 EndAbility로 종료해야 합니다.
`OnWeaponFireFinished` 결과를 받은 뒤 EndAbility를 호출하고, 활성화 실패 경로도 종료하는지 확인합니다.

### ExitState 노드와 핀

`Event ExitState → Branch(bAbilityStarted)`:

- false: 추가 실행 없음.
- true: `Set bAbilityStarted=false → Is Valid(Context.StateMediator) → Cancel Ability By Tag`.
  - Target = EntityContext.StateMediator
  - Ability Tag = Ability.Weapon.Fire

정상 완료는 Tick에서 bAbilityStarted를 먼저 false로 바꾸므로 정상 Exit에서 재취소하지 않습니다.
GA의 취소/EndAbility 경로에서 몽타주와 무기 실행을 정리합니다. 두뇌 Attack 이탈의 C++ 정리도 멱등하게 수행됩니다.

## 5. Upper의 Ready 복귀

- Upper Attack의 `On State Completed → Ready`를 설정합니다. 성공·실패 모두 Ready로 복귀할 때 이 전이를 사용합니다.
- Ready Task의 EnterState에서 `Publish Upper State(ReadyEntered)`가 반드시 실행되어야 합니다.
  - Target = NPC의 EntityContext.StateEventBus
  - Source = NPC EntityCharacter
- Reload 완료·실패 뒤에도 Ready Task에 진입하고 동일 이벤트를 발행합니다.
- CarryState가 Ready인 것만으로 이벤트를 대체하지 않습니다. 반복 요청은 Upper의 실제 복귀 알림을 기다립니다.
- Base 트리의 ChaseStarted 명령 → Chase 상태 / CancelChase 명령 → Idle 연결은 유지합니다.
- `STT_NPC_ChaseWalk`가 NPC Controller의 CurrentTarget을 AI MoveTo Target Actor로 사용하고 ExitState에서 StopMovement를 수행하는지 확인합니다.

## 6. 검증할 장면

| 장면 | 기대 결과 |
|---|---|
| NPC 뒤/옆에서 공격 거리 진입 | 몸이 설정 각속도로 회전하고 허용 오차 안에 들어온 뒤 첫 발 |
| 사거리 안에서 계속 대기 | GA 정상 종료 → Upper Ready 복귀 → 다음 공격. 쿨다운을 무시하지 않음 |
| 사거리 안에서 타겟이 옆으로 이동 | 몸이 따라 회전. 방향이 틀어진 진행 중 공격은 정리하고 재정렬 후 재요청 |
| 사거리 밖으로 이동 | 두뇌 Attack → Chase, Base Chase 진입, MoveTo 재개 |
| 다시 사거리 안으로 진입 | Chase 종료 → 회전 정렬 → 공격 반복 재개 |
| 탄창 소진 | Reload 요청 → Reload GA → Ready → 공격 재개 |
| GA 활성화 거절 | Task 실패 → Ready. 성공으로 오인하거나 매 Tick GA 활성화를 반복하지 않음 |
| 타겟 포기/사망/UnPossess/PIE 종료 | 공격 입력·GA·몽타주·Focus·EventBus 구독 정리 |

로그·PIE 증거를 확인하기 전까지 세 버그의 해결 검증은 열린 상태입니다.

## 전체 실행 순서

```text
Chase: NPC Set Brain State(Chase)
 → NPC Set Chase Command(true) → 타겟 Focus / Base Chase 명령
 → STT_NPC_ChaseWalk → AI MoveTo(TargetActor=CurrentTarget)
 → 사거리 안 Condition 통과 → Attack

Attack: NPC Set Brain State(Attack)
 → NPC Set Chase Command(false) → Base CancelChase → MoveTo 중지
 → NPC Combat Attack Enter → Focus / EventBus 구독 → Running
 → Task Tick → 타겟·사거리·사망·GA·무기 상태 검사
   ├─ 몸 방향 불일치 → 진행 중 공격 정리 → 회전 정렬 대기
   ├─ 수납 상태 → Draw만 요청 → ReadyEntered 대기
   ├─ 공격/재장전 중 → 종료 및 ReadyEntered 대기
   └─ 방향 일치 + Ready + 실행 가능
      → AttackStarted 의미 입력 (빈 탄창이면 Reload 입력)
      → Entity InputBuffer 소비 → Mediator → Upper 전이
      → STT_NPC_Attack Enter
        → Publish FireEntered → TryActivateAbilityByTag(한 번)
          ├─ false → FinishTask(false) → Ready → Publish ReadyEntered
          └─ true → bAbilityStarted=true → GA_Attack
             → Weapon Fire / Montage / Notify
             → OnWeaponFireFinished → EndAbility
             → Tick에서 Attacking 태그 해제 확인
             → bAbilityStarted=false → FinishTask(DidLastFireSucceed)
             → Ready → Publish ReadyEntered
             → 두뇌 Task가 다음 공격 가능 상태 확인 → 재요청

중단: 사거리 밖 / 타겟 상실 / 사망 / 트리 종료
 → NPC Combat Attack Exit 또는 Controller/Pawn 정리
 → 공격 종료 입력 / 공격 버퍼 제거 / Fire GA 취소 / Fire 실행 정리
 → Focus 해제 / EventBus 구독 해제
 ├─ 사거리 밖 → Chase 진입 → 타겟 Focus 재설정 → MoveTo 재개
 ├─ 타겟 없음 → Idle
 └─ 사망 → Dead

Upper Task 자체의 취소 Exit
 → bAbilityStarted=true인 경우에만 false로 전환 → CancelAbilityByTag(Fire)
 → 이후 Ready 또는 해당 중단 상태로 복귀
```
