# 공통 사망 GA·몽타주·제거 연결

2026-10-25 보스 후속 사용자 해결 확인: 체력 0·BrainState 비전투·Idle 모션 잔류 사례는 보스 Death 전이를 `Priority=Critical`로 수정해 해결. 공격 중 사망을 금지하는 정책이나 일반 피격 보호 옵션의 영향으로 기록하지 않음. 사용자 에셋 수정·플레이 확인이며 Codex가 빌드/PIE 실행한 증거와 구분

2026-10-21 사용자 확인: 사망 전이 문제 해결·사망 우선권 상향 유지. STT_EntityDeath는 STT_EntityBase 자식. 아래 C++·에디터 연결 기준 유지, 개별 무기 조합 전체 검증으로 확대하지 않음

2026-10-22 C++ 반영: 사망 에셋을 캐릭터 BP의 단일 속성에서 현재 ABP의 DeathMontages 배열로 이전. 무작위 선택·기존 완료·제거 경로 정적 대조. 최신 빌드·Editor 설정·PIE 대기

## 설계 의도

- 사망 실행 지시는 Base StateTree의 Death Task. Player·NPC 공통 Task·GA 사용
- HealthComponent는 사망 사실·State.Dead 확정. Mediator는 기존 Ability 취소 후 Upper 비활성화·Base Death 명령 전달
- Death Task는 GA 활성화 요청, GA는 실행 시작·결과·종료 처리, EntityCharacter는 실제 재생·완료·제거 단일 소유
- Mediator의 사망 GA 직접 활성화·C++ 우회 실행 제거. 연결 실패는 오류로 노출
- Base는 Death 상태를 Actor 제거 시점까지 유지. NPC에는 별도 Brain 정지·전리품 생성 훅만 유지
- 블렌드아웃 시작 시 제거. 기본 서 있는 포즈 복귀 후 사라지는 구간 제거. 추가 대기 Timer 없음
- Player도 사망 연출 후 제거. 부활·재시작·카메라 후속 정책은 이번 범위 제외

## 설정 위치

각 캐릭터 Mesh에 연결한 ABP의 Class Defaults → Animation | Actions → Action Animations → Death Montages 배열

1. 기존 캐릭터 BP의 Death Montage 값을 빌드·에디터 재시작 전에 기록
2. 각 ABP의 Death Montages 배열에 해당 스켈레톤의 사망 몽타주를 한 개 이상 등록
3. 모든 항목의 슬롯을 기존 최종 FullBody 경로에 맞게 지정
4. ABP·캐릭터 BP Compile 후 Player·NPC 각각 사망 확인

기존 사용 에셋: `/Game/Astra/Entity/Humanoid/Animations/Flying_Back_Death_Montage.Flying_Back_Death_Montage`. 다른 스켈레톤의 ABP에는 호환·리타겟한 몽타주 지정

사망 시작 시 유효 배열에서 하나 무작위 선택 후 ActiveDeathMontage에 보관. 배열 한 개도 허용, 사망 간 같은 항목의 반복 선택 허용. 같은 사망의 반복 요청은 재선택 없음. 빈 배열·None 항목은 Error 로그 후 기존 재생 실패 완료·제거 경로

EntityCharacter.DeathMontage 속성과 해당 속성을 대상으로 한 과거 NPC PropertyRedirect 제거. 캐릭터 BP→ABP 배열의 소유 객체·자료형 변경으로 자동 값 이전 없음. C++ 에셋 경로 하드코딩 없음

## Base StateTree와 공통 Death Task

초기 ST_PlayerBaseState 내보내기는 Root 아래 GroundState·AirborneState·HangingState 구성, Death 미포함. 후속 NPC Base 내보내기에서 Death·공통 Task·Root 전이 확인. 사망 전이 해결·우선권 상향 사용자 확인. 아래 절차는 구성 기준

### Death 상태와 명령 전이

1. Root의 직접 자식으로 Death 추가. GroundState·AirborneState·HangingState와 같은 깊이. 기본 초기 선택 대상 변경 방지를 위해 기존 자식 뒤에 배치
2. Root에 공통 전이 추가
   - Trigger: On Event
   - Event Tag: `StateTree.Event.LocomotionCommand`
   - Payload Struct: `LocomotionStateCommandPayload`
   - Condition: Compare Enum, Left는 해당 이벤트 Payload.Command 바인딩, Right는 `ELocomotionStateCommand::Death`
   - Target: Death
   - Priority: Critical
3. Death에 공통 Blueprint Task `STT_EntityDeath` 배치
4. Death에서 Idle·Root·GroundState로 돌아가는 완료·실패·Tick 전이 없음. 반복 Death 명령에 의한 재진입 전이 없음
5. Root의 다른 공통 전이가 있다면 State.Dead 동안 일반 행동 전이 차단. Death 상태 Tag 속성으로 ASC.State.Dead 대체 금지

### STT_EntityDeath

사용자 확인 구조: `StateTreeTaskBlueprintBase → STT_EntityBase → STT_EntityDeath`

- 직접 부모 클래스: STT_EntityBase. C++ Task 추가 없음
- STT_EntityBase의 상속 변수: actor, character, entitycontext, pawn. StateTree에서 참조 바인딩
- 바인딩된 entitycontext.StateMediator를 직접 사용. Actor 캐스팅·GetContext 재호출·DeathMediator 별도 저장 없음
- 필수 바인딩 누락은 오류 확인 후 StateTree 연결 수정. IsValid 분기로 실행을 생략하는 방어 처리 없음
- Should State Change On Reselect: false
- ReceiveTick 없음. GA 활성화 재시도·프레임별 호출 없음
- Death Task는 제거 시점까지 Running 유지. 활성화 요청 성공을 FinishTask(true)로 변환하지 않음
- ReceiveExitState 추가 처리 없음. GA 취소·바인딩 해제·참조 초기화는 Death Task의 책임에서 제외. GA·C++의 종료 처리 사용

Enter 이벤트는 UE 5.8의 반환값 없는 `Event EnterState` 사용. 내부 함수명 `ReceiveLatentEnterState`. `EnterState (Deprecated)` 함수의 `ReceiveEnterState`와 구분. 부모에 새 Enter 이벤트 구현이 있으면 구형 Enter 함수 대신 새 이벤트만 호출하는 엔진 분기 확인. 같은 이름의 Custom Event로 대체 금지

```text
Event EnterState (ReceiveLatentEnterState)
→ Try Activate Ability By Tag
   Target=상속받아 바인딩된 entitycontext.StateMediator
   AbilityTag=Ability.Entity.Death
   ├─ true → 실행 요청 수락, 그대로 Running 유지
   └─ false → 활성화 실패 오류 출력, 그대로 Running 유지

Task 이탈 → 추가 작업 없음
```

진입 시 Context·Mediator 누락은 구성 오류. 오류·로그를 직접 확인하고 바인딩 원인 수정. Task 진입·이탈에서 Pawn·Controller·Context 재조회와 불필요한 IsValid 분기 없음

활성화 실패도 FinishTask(false) 호출 없음. 부모의 완료 처리로 Root 재선택·GA 반복 활성화 방지. 실패는 로그로 보고, 사망 상태 유지. 기존의 일반 Activate Entity Ability By Tag C++ Task로 대체 시 즉시 Succeeded/Failed 반환 차이 주의

TryActivateAbilityByTag 정책: State.Dead 없는 Death 요청 거부. State.Dead 중 일반 Ability 거부. Death 요청만 기절 차단 예외. 이후 ASC의 GA Required/Blocked Tags 검사 유지

### 해결 기록: Death Task 미진입·Patrol 재진입

사용자 확인 순서: Player·NPC의 GA_EntityDeath 등록 누락 발견·추가 → Death Task 진입·GA 요청 실행 없음과 Patrol 출력 관측 → 전이 문제 수정 후 해결 보고 → 사망 우선권 상향 유지 확인. 미해결 진단 대신 해결 기록으로 관리

- 제공된 ST_NPCBaseState 텍스트의 Root→Death 전이 우선순위는 기본 Normal. 설계값 Critical과 불일치
- UE 5.8 TriggerTransitions는 이미 선택된 전이보다 우선순위가 같거나 낮은 후속 전이를 생략. 같은 프레임의 일반 이동 전이에 사망 전이가 밀릴 가능성
- Root→Death의 Priority를 Critical로 수정. Left는 해당 Death 전이 자신의 Event.Payload.Command로 다시 바인딩, Right는 ELocomotionStateCommand.Death
- GroundState의 공통 알림 Task가 첫 Tick 완료 방식이면 Considered for Completion 해제. 부모 상태 완료로 일반 상태 재선택 방지
- 현재 UpdateTargetTracking은 사망 확인 시 타겟 정리, ForgetCurrentTarget은 BrainState.Idle 설정. DamageLook의 사망 중 Update는 완료 반환. Death 선점 실패 시 일반 복귀 조건 성립 가능
- Patrol 출력에는 바인딩된 character의 이름 함께 표시. 복수 NPC의 다른 Task 로그를 같은 NPC 사망 흐름으로 혼동하지 않음
- Death 활성 상태와 Death Task 첫 출력 확인 전 GA 태그·몽타주 문제로 범위 이동 금지
- 같은 상태의 전이 목록 순서는 동순위 평가 순서. 활성 자식부터 부모 방향의 평가와 별개. Root Death를 목록 맨 위로 배치하는 것만으로 자식 전이보다 우선 보장 불가. 사망 Priority=Critical 유지

```text
같은 NPC의 체력 0
→ Root Death 전이: Critical / 자기 Event.Payload.Command == Death
   ├─ 미선택 재발 시 → 이벤트 발신·바인딩·활성 상태 확인
   └─ 선택 → STT_EntityDeath.Event EnterState 첫 출력
      → GA 요청 결과 확인

정상 완료·재생 실패·연출 중단 → 공통 사망 완료·제거
일반 Patrol 복귀 → 사망 처리 성공으로 판단하지 않음
```

## 공통 GA Blueprint

기존 사망 GA가 있다면 같은 에셋 수정. 없다면 GameplayAbility 기반 공통 `GA_EntityDeath` 생성

- Instancing Policy: Instanced Per Actor
- Net Execution Policy: Server Only. 현재 사망 판정·제거 권한 경로 기준
- Asset Tags / Ability Tags: `Ability.Entity.Death`
- Activation Required Tags: `State.Dead`
- Activation Blocked Tags에 State.Dead·State.Stunned 또는 이를 포함하는 상위 State 태그 포함 금지. 확정된 사망은 기절로 차단하지 않는 정책
- Activation Owned Tags에서 State.Dead를 중복 소유하지 않음. HealthComponent 소유 유지
- Player·NPC의 StartupAbilityClasses에 같은 GA 클래스 등록. Base Death Task 한 곳에서 한 번 활성화 요청. Upper·Brain·Health·Mediator의 중복 활성화 제외
- GA에서 Montage Play·Delay·DestroyActor·전리품 생성 없음

GA 멤버 `DeathEntity`: EntityCharacter Object Reference. 로컬 변수 대신 수명 동안 보관할 멤버 사용

완료 Custom Event는 Bind의 빨간 Event 핀에서 Add Custom Event로 생성. 이름 `HandleDeathPresentationFinished`, 입력 `bInterrupted` Bool. Bind와 Unbind에 같은 Event의 델리게이트 출력 연결. 새 Gameplay Event 태그 불필요

ActivateAbility:

```text
Get Avatar Actor from Actor Info → Cast To EntityCharacter
├─ 실패 → 오류 출력 → CancelAbility
└─ 성공 → DeathEntity 저장
   → Bind Event to OnDeathPresentationFinished
      Target=DeathEntity, Event=HandleDeathPresentationFinished
   → BeginDeathPresentation(Target=DeathEntity)
      ├─ true → 완료 Event 대기
      └─ false → 오류 출력 → CancelAbility
```

완료 Event(bool bInterrupted): EndAbility. 사망 연출 중단도 확정된 사망 완료로 처리. 알림 발신이 돌아온 뒤 EntityCharacter가 CompleteDeath 수행. 별도 CompleteDeath·DestroyActor 호출 불필요. 몽타주 미지정·재생 실패는 BeginDeathPresentation 안에서 동기 완료 가능하므로 반드시 호출 전 Bind

OnEndAbility: 유효한 DeathEntity에서 자기 완료 Event만 Unbind → DeathEntity=None. 무효 참조도 DeathEntity=None. 해제 처리는 이곳으로 통일. Unbind All로 다른 구독자 제거 금지. 정상 End에서 사망 몽타주 Stop·GA 재활성화 없음

GA 시작 실패: 오류 보고·CancelAbility, Base Death 유지. Mediator·Task에서 BeginDeathPresentation을 직접 호출하는 우회 없음. GA 시작 후 외부 취소: 바인딩 해제, 이미 시작된 C++ 사망 연출·제거 계속 수행. 취소로 생존 상태 복원 없음

## AnimGraph와 Slot

ABP_Unarmed의 기본 Slot은 FinalBasePose 이전. 이후 spine_01 기준 Layered Blend per Bone에서 무기 상체 포즈를 덮는 구조 확인

1. Skeleton Anim Slot Manager의 DefaultGroup 아래 FullBody Slot 생성
2. 사망 몽타주의 Slot을 DefaultGroup.FullBody로 지정
   - Enable Auto Blend Out 켬. 마지막 Section의 반복 연결 없음. 블렌드아웃 시작 사건으로 정상 완료 확보
3. 기존 최종 Component To Local Space와 Output Pose 사이에 Slot(DefaultGroup.FullBody) 추가
4. 이 Slot 이후 상체 Layered Blend·AimOffset·척추 Modify Bone 배치 없음
5. 재장전·사격은 기존 UpperBody Slot 유지

```text
이동·기본 Slot → FinalBasePose
→ 무기 상체 합성 → 척추 회전 보정
→ Component To Local Space
→ Slot(DefaultGroup.FullBody)
→ Output Pose
```

AstraHumanoidAnimationData.bIsDead 추가. 사망 시 bWeaponUpperBodyActive·bEnableWeaponAimOffset 해제, 무기 척추 보정 0. AnimGraph는 읽기만 수행

## 실행과 종료

BeginDeathPresentation은 사망 사실·권한 재검사 후 중복 실행 방지. 입력·무기 실행·이동 정지, Upper 정지, NPC의 Brain 정지 후 몽타주 재생. Base 정지 없음. Actor Tick에서 사망 중 무기 입력 처리 없음

블렌드아웃 시작·몽타주 종료 중 첫 알림만 완료 처리. 메시 애니메이션 갱신 정지 → GA 완료 알림 → NPC 전리품 훅 → 공통 Destroy 순서. 성공·재생 실패·중단 모두 동일 종료 경로

기존 Death Task는 공통 Task로 통합. Task의 Montage Play·DestroyActor·BeginDeathPresentation 직접 실행 제거. GA도 Montage Play·DestroyActor 중복 실행 없음

## 검증 항목

- Player·NPC 각각 HP 0 → Base Death 진입·Task 한 번 요청·같은 GA·각 ABP의 배열에서 한 개 선택·전신 재생·제거
- 기절 중 사망 → Death GA 허용. 생존 중 Death 요청·사망 중 일반 Ability 요청 거부
- 사망 GA 미등록·차단 → 오류 노출·Death 유지·일반 행동 복귀·반복 활성화·C++ 우회 없음
- BeginDeathPresentation 이후 Base Death Task 유지. Upper·NPC Brain 정지
- 권총·라이플·비무장, 공격·재장전·Sprint 중 사망
- 블렌드아웃 때 서 있는 포즈 노출 없이 제거
- NPC 전리품 한 번 생성. Player는 전리품 생성 없음
- 미지정·재생 실패·중단 시 오류·완료 처리 후 잔존 Actor 없음
- 반복 피해에도 GA·몽타주·전리품·제거 중복 없음
- 각 캐릭터 ABP의 DeathMontages 지정, 여러 번 실행 시 배열 항목 범위·1회 선택 확인

## 전체 순서

```text
체력 0 → Health.State.Dead·사망 사실
→ Mediator: 자동 재장전 요청 폐기·기존 Ability 취소
→ Upper CancelAction·Disable / Base Death 명령
→ Base Root의 Death 이벤트 전이 → Death 상태 유지
→ STT_EntityDeath.Event EnterState → Mediator.TryActivateAbilityByTag(Ability.Entity.Death)
   ├─ 거부 → 오류 출력 → Death 유지, 재요청·C++ 우회 없음
   └─ 수락 → GA: Entity 확보·완료 Event Bind → BeginDeathPresentation
      ├─ 권한·사망 사실 불충족 → false → 오류·CancelAbility → 구독 해제
      ├─ 몽타주 미지정·재생 실패 → 오류·즉시 완료
      └─ 몽타주 재생
         ├─ 블렌드아웃 시작 → 완료
         └─ 중단·종료 → 아직 완료하지 않은 경우 완료

완료 → 메시 갱신 정지·콜백 해제
→ OnDeathPresentationFinished → GA.EndAbility → OnEndAbility에서 자기 Event 해제
→ CompleteDeath 한 번
   ├─ NPC → 전리품 생성
   └─ Player → 추가 전리품 없음
→ 공통 Destroy

GA 외부 취소 → OnEndAbility에서 자기 Event 해제 → 시작된 사망 연출·공통 제거 계속 수행
Task 이탈 → 별도 처리 없음
외부 Actor 제거·레벨 종료 → GA·ASC와 Entity의 생명주기 종료 처리
→ GA.OnEndAbility에서 자기 Event 정리, Entity.EndPlay에서 몽타주 콜백 해제
```
