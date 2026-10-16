# NPC Idle 순찰·피격 방향 확인 연결

2026-10-08 C++ 반영. 최신 재빌드·재실행 후 발사 정상 사용자 확인. 순찰 Blueprint 연결 진행. HitReaction 상태 미구성 사용자 확인

## 설계 의도

- 두뇌는 Idle 진입·이탈 명령 담당. 실제 순찰·방향 확인은 Base의 Blueprint Task에서 C++ API 호출
- 2026-10-09 사용자 요청 반영: Idle·Patrol 진입 시 수납 요청 제거. 두뇌 Idle Task에서 무기 활동 갱신으로 자동 수납 방지
- 피격 순간의 공격자 위치만 방향 정보로 저장. 시야 확인 전 CurrentTarget·마지막 확인 위치 기억 생성 금지
- 기존 추적·공격·재장전 경로 유지

## DA 설정

기존 NPC EntityDataAsset의 Combat Tendency Data에 설정 추가. 별도 DataAsset 생성 불필요

| 설정 | 용도 | 코드 기본값 |
|---|---|---|
| bEnableIdlePatrol | Idle 순찰 사용 | false |
| PatrolRadius | 스폰 위치 기준 수평 순찰 반경 | 0 |
| PatrolWaitTime | 순찰 도착 후 대기 시간 | 0 |
| PatrolAcceptanceRadius | 순찰 도착 허용 거리 | 50 |
| PatrolSelectionAttempts | 한 번의 목적지 선택에서 경로 검사 횟수 | 5 |
| DamageLookWaitTime | 피격 방향 회전 완료 후 확인 대기 시간 | 0 |
| DamageLookRotationRate | 피격 방향 회전 속도, 도/초 | 360 |
| DamageLookToleranceYaw | 회전 완료 허용 각도 | 5 |

순찰 사용 시 bEnableIdlePatrol 체크 및 PatrolRadius 양수 지정 필수. 반경·대기 시간의 게임플레이 값은 사용자 지정

목적지는 스폰 위치 주변의 도달 가능한 NavMesh 지점에서 선택. 현재 위치에서 목적지까지 완전 경로 검사. 순찰 범위 안에서 시작하는 경로는 경로 점도 반경 안으로 제한. 전투 종료 후 범위 밖에서는 범위 안 목적지로 복귀 경로 허용

목적지 선택·이동 실패 시 기존 ChaseRetryDelay 간격으로 재선택. 매 프레임 이동 요청 덮어쓰기 금지. 스폰 위치 중심 유지, 매 Idle 진입 시 중심 재설정 없음. 같은 Pawn 재Possess 시 중심 유지

## NPCBrain Idle

1. 기존 Idle의 상태 설정·추적 종료·공격 종료 Task 유지
2. 마지막에 새 네이티브 Task **NPC Idle Command** 추가
3. AIController에 두뇌 Context 바인딩
4. Considered for Completion 체크. 해당 Task는 Idle 동안 Running 유지
5. 기존 Root의 NPC Update Target Tracking 유지
6. 기존 타겟 기억·사거리 전이 유지. Attack → Idle의 기억 조건 Invert 체크

NPC Idle Command는 AIController Context의 NPCBrain Idle 전용. Base Idle에 배치 시 Context 바인딩 오류

진입 명령: PatrolStarted → NPCCharacter → Mediator.Patrol. 무기 수납 요청 없음

이탈 명령: PatrolCompleted → NPCCharacter → Mediator.CancelPatrol

NPC Idle Command의 Enter·Tick에서 UpdateIdleWeapon 호출. 기존 Ready 무기의 자동 수납 타이머 갱신. 별도 Draw 요청·Montage Play·강제 부착·CarryState 직접 변경 없음

## Base 트리

새 Blueprint Task 이름은 아래 안내용 이름 사용 가능. 실제 프로젝트 명명 규칙에 맞춰 생성

- Patrol 상태 추가. 기존 Idle 계층의 자식으로 구성 가능
- LocomotionCommand 이벤트 Payload의 Command=Patrol → Patrol 상태 연결
- Command=CancelPatrol은 Patrol 상태에서만 기존 Idle로 연결
- Patrol→ChaseWalk에 Command=Chase 전이 추가. 해당 전이 자신의 Payload.Command 바인딩
- 기존 Chase·CancelChase 전이는 Chase 관련 상태로 범위 제한
- HitReaction 명령은 Patrol·Chase·정지 상태에서 피격 방향 확인 상태로 연결
- 피격 방향 확인 상태에서 HitReaction에 대한 자기 재진입 제외. 연속 피격의 새 방향은 C++ Tick에서 반영
- 사망 전이를 최우선 유지

### STT_NPC_Patrol: Blueprint

변수 NPCController: AstraNPCAIController Object Reference. 기존 부모의 Pawn 바인딩 재사용

EnterState:

```text
Event EnterState → Get Controller(Target=바인딩된 Pawn)
 → Cast To AstraNPCAIController
    ├─ Failed → 구성 오류 로그 → Finish Task(Succeeded=false)
    └─ Success → Set NPCController → Request Idle Patrol(Target=NPCController)
                 → Running 유지
```

Tick:

```text
Event Tick → 저장된 NPCController.Update Idle Patrol → Running 유지
```

반환 enum은 디버그 출력용. Waiting·Moving·Failed에 대한 Finish Task 호출 없음. C++에서 도착 대기 및 다음 목적지 선택 처리

Enter마다 Controller 확보. Actor 바인딩 사용 시 Get AIController의 반환값을 Cast에 연결. Actor를 Controller로 직접 Cast하지 않음. 필수 Controller 누락을 성공 처리로 숨기지 않음

ExitState:

```text
Event ExitState → IsValid(NPCController)
 ├─ Valid → Cancel Idle Patrol(Target=NPCController) → Set NPCController=None
 └─ Invalid → Set NPCController=None
```

Considered for Completion 체크. 별도 AI MoveTo·BP Timer·목적지 난수 계산 중복 추가 금지. Task 참조 실패 시 Base Idle로 복구 후 다음 Patrol 명령에서 재시도

### STT_NPC_DamageLook: Blueprint

현재 Base HitReaction 상태 없음 사용자 확인. 상태와 방향 확인 Task 신규 연결 필요. 별도 피격 몽타주 Task가 있는 경우 완료 정책 All로 두 작업 완료 후 이탈

변수 NPCController: AstraNPCAIController Object Reference

EnterState:

```text
Event EnterState → IsValid(Pawn)
 ├─ Invalid → Finish Task(Succeeded=false)
 └─ Valid → Get Controller → Cast To AstraNPCAIController
            ├─ Failed → Finish Task(Succeeded=false)
            └─ Success → Set NPCController → Begin Damage Look → Branch(Return Value)
                         ├─ true → Running 유지
                         └─ false → Finish Task(Succeeded=true)
```

Begin Damage Look=false는 방향 정보 없는 피해 등 회전 작업 생략 경로. 임의 방향 생성 없음

Tick:

```text
Event Tick → IsValid(NPCController)
 ├─ Invalid → Finish Task(Succeeded=false)
 └─ Valid → Update Damage Look(Target=NPCController, DeltaTime=Event Tick의 DeltaTime)
            → Branch(Return Value)
               ├─ true → Finish Task(Succeeded=true)
               └─ false → Running 유지
```

ExitState:

```text
Event ExitState → IsValid(NPCController)
 ├─ Valid → End Damage Look(Target=NPCController) → Set NPCController=None
 └─ Invalid → Set NPCController=None
```

Considered for Completion 체크. 정상 완료 후 Controller.GetBrainState 기준 복귀 목적지 분리

| 두뇌 상태 | Base 복귀 |
|---|---|
| Idle | Patrol |
| Chase | 기존 Chase |
| Attack | 기존 정지 상태 |
| 사망·전투불능 | 기존 사망·전투불능 상태 |

피격 방향 확인 상태에서 Chase·Patrol 명령 즉시 전이 제외. 회전 완료 후 현재 두뇌 상태로 복귀. 두뇌 Attack Task가 회전 진행 중 추가 발사 요청을 대기하도록 C++ 처리

실패 시 정상 완료와 별도 복구 전이 구성. NPCController가 유효하면 위 두뇌 상태 기준 복귀, 참조가 없으면 안전한 정지 상태로 복귀. 사망·취소 시 ExitState를 통한 요청 정리

## 피격 정보와 제한

기존 HealthComponent.OnDamageTaken → Mediator에서 상체 행동 취소 → 피해 방향 요청 중계 → Base HitReaction 명령 전달

NPCCharacter가 방향 요청을 받아 Controller.RememberDamageDirection 호출. DamageSource의 Instigator가 있으면 해당 위치, 없으면 DamageSource 위치 사용. 피격 순간 좌표 저장 후 회전 중 공격자 실제 위치 재조회 없음

DamageSource 누락·자기 자신·동일 위치 피해는 방향 확인 생략. 실제 탄환 궤적 역방향 계산은 이번 범위에서 제외

## 사격 속도 확인

몽타주 미재생 문제가 아닌 발사 간격 조정 요청으로 정정

- 기존 무기 DA의 AttackCooldown이 발사 후 쿨다운 담당. 코드 기본값 0.2초, 실제 에셋 값 미확인
- FireMontage의 Weapon_FireCommit 시점에서 실제 발사 및 쿨다운 시작
- 몽타주·GA 종료와 쿨다운 상태에 따라 실제 다음 발사 시점 결정
- AttackRequestRetryDelay는 거부·미소비 요청 재시도 간격. 사격 속도 튜닝 항목으로 사용 금지
- 같은 무기 DA를 Player·NPC가 공유하면 AttackCooldown 변경도 공통 적용
- 사용자 확인: 무기 DA의 AttackCooldown=1초. 시야 상실·재발견 시 발사 간격 무시 증상
- 원인 코드: 연발형 CancelFire 및 FinishFire에서 쿨다운 타이머 제거. 공격 취소 후 재진입 시 1초 제한 소실 가능
- 수정: 공격 취소·입력 종료·Sequence 종료에서 발사 쿨다운 유지. 타이머 만료 또는 무기 런타임 초기화에서만 해제
- CanAttackWeapon에 쿨다운 검사 추가. 상태가 Idle이어도 쿨다운 중 새 발사 거부
- NPC 전용 추가 대기 설정 없음. 기존 DA 1초 값 유지. Player·NPC 공통 발사 경로에 적용
- 후속 보완: Attack 재진입 시 CarryState.Ready로 준비 여부 초기화. 쿨다운의 ActionState.Attacking과 준비 신호 분리. 발사 권한은 ActionState·GameplayTag·쿨다운 검사에서 별도 판단
- 피격 방향 정보 대기만으로 발사 중단 금지. 실제 DamageLook Task 활성 구간에서만 추가 발사 대기

## 첫 발 미발사 진단

사용자 확인: 후속 변경 재빌드·재실행 후 발사 정상. 첫 발 미발사 증상의 단일 원인은 미확정

후속 C++ 보완: Ready 초기화와 쿨다운 분리, 미실행 피격 방향 요청의 발사 차단 제거, 공격 대기 원인 진단 로그 추가. 순찰·피격 방향 에셋 연결 완료 확인은 미기록

재빌드 후 PIE 시작 전에 Output Log 명령 입력: `astra.NPC.CombatDebug 1`

NPC에 접근 후 `NPCCombat` 포함 로그 확인. 원인 변경 시에만 출력. 종료 후 `astra.NPC.CombatDebug 0`으로 비활성화

| Reason | 확인 위치 |
|---|---|
| BeginRejected / MissingEventBus | Controller·Mediator 초기화·타겟 시야 |
| DamageLookActive | 피격 회전 Task의 Tick·완료·Exit |
| TagOrActionBlocked | 공격 권한·차단 태그·무기 Action |
| WeaponNotReady | Upper Draw·Holster 경로 |
| FacingTarget | NPC 몸 방향 정렬 |
| AwaitUpperReady | ReadyEntered 발행 |
| BufferedInputUnconsumed | Upper의 공격·재장전 입력 소비 |
| WeaponCannotAttack | 무기 쿨다운·탄약·공격 조건 |
| AttackRequested | 요청 발신 완료, 이후 Upper·GA 실행 경로 |

## 확인 항목

1. 시작 Idle·Patrol: 수납 요청 없이 무기 휴대 유지. 자동 수납 지연 시간 경과 후에도 수납되지 않는지 확인
2. 순찰: 지정 반경·NavMesh 경로·도착 대기·다음 목적지 선택
3. 경로 부재: 재선택 대기. 매 프레임 MoveTo 발신 없음
4. 후방 피격: 순찰 중단 → 피격 순간 방향으로 회전 → 감지 후 Chase·Attack
5. 벽 뒤 공격자: 회전 후에도 시야 미확보 시 타겟 신규 획득 없음
6. 연속 피격: 최신 피격 방향 반영. Task 자기 재진입으로 방향 요청 소실 없음
7. 전투 중 피격: 회전 Task 동안 추가 발사 대기, 완료 후 두뇌 상태에 맞는 Base 복귀
8. 타겟 기억 만료: 기존 Idle 전이 → 무기 휴대 유지 → 순찰 재개
9. 사망·UnPossess·Controller 종료: 순찰·추적·피격 방향 요청 정리
10. AttackCooldown=1초에서 첫 발 직후 1초 이내 시야 상실·재발견: 다음 탄 발사 간격 1초 이상 확인
11. 연발 입력 해제·재입력, 방향 정렬 중 취소, 피격·수납·재장전 후 재발사: 같은 쿨다운 유지 확인
12. 첫 발 확정 전 취소: 탄약 소비·새 쿨다운 생성 없음. 기존 쿨다운 만료 후 정상 발사 확인
13. Player 단발·연발 및 NPC 반복 공격: 정상 발사·종료 알림·쿨다운 만료 후 Ready/Idle 복귀 확인
14. 쿨다운 중 재발견: 새 ReadyEntered 없이도 기존 쿨다운 만료 후 발사 재개 확인
15. 피격 방향 Blueprint 미연결 상태: 방향 정보 대기만으로 영구 발사 정지 없음 확인

## 전체 실행 순서

```text
NPCBrain Idle Enter → NPC Idle Command
 → PatrolStarted → NPCCharacter
    ├─ Enter·Tick.UpdateIdleWeapon → 무기 활동 갱신 → 자동 수납 방지
    └─ Mediator.Patrol → Base Patrol Enter → RequestIdlePatrol
       → Tick.UpdateIdlePatrol → 도달 가능한 목적지·완전 경로 선택
          ├─ 성공 → 이동 → 도착 → PatrolWaitTime → 다음 목적지
          └─ 실패 → ChaseRetryDelay 대기 → 제한 횟수로 목적지 재선택

피격 → Health 이벤트 → Mediator
 → Upper CancelAction → NPC에 피격 방향 전달 → Base HitReaction
 → 기존 순찰·추적 Exit 및 이동 취소
 → DamageLook Enter → BeginDamageLook
    ├─ 방향 없음 → 정상 완료 → 현재 두뇌 상태로 복귀
    └─ 방향 있음 → Tick.UpdateDamageLook → 회전 → 확인 대기
       ├─ 시야 감지 → 기존 두뇌 Chase·Attack → 완료 후 Base 상태 복귀
       └─ 미감지 → 대기 종료 → 기존 두뇌 상태로 복귀

Idle 이탈 → NPC Idle Command Exit → CancelPatrol
 → Patrol Exit → CancelIdlePatrol
참조 실패 → FinishTask(false) → Base 복구 전이 → Exit 정리
사망·취소 → Task Exit → 해당 이동·회전 요청 정리
UnPossess·Controller 종료 → ForgetCurrentTarget
 → 순찰·피격 방향·공격·추적·기억 정리
```
