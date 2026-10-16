# NPC 착지 후 현재 의도 복구 연결

2026-10-09 C++ 반영. 사용자 빌드·Blueprint 연결·PIE 검증 대기

## 상태 책임

| 소유자·상태 | 책임 |
|---|---|
| Controller BrainState | 현재 행동 의도 보관 |
| Brain Idle | 평상시 순찰·대기 의도 |
| Base Idle | 이동 요청 없는 상태. 제자리 사격 포함 |
| Base Patrol | 순찰 이동·도착 후 대기 반복 |
| Base ChaseWalk | 타겟·마지막 확인 위치 접근 |
| Base Falling | 공중 물리 상태. 기존 이동 실행 중단 |
| Upper | 무기 꺼내기·수납·발사·재장전 실행 |

Falling 진입 시 BrainState·타겟 기억 초기화 금지. 착지 시 이전 Base 상태 대신 현재 BrainState 사용. Idle·Patrol에서 수납 요청 없음

## C++ 연결

`GroundedEntered → EventBus → Mediator.OnGroundedReady → NPCCharacter → Controller.ReissueCurrentLocomotionIntent`

| 현재 BrainState | 재발행 명령 |
|---|---|
| Idle + 순찰 사용 + 양수 반경 | Patrol |
| Idle + 순찰 비활성화·반경 0 | Stop |
| Chase + 타겟 기억 | Chase |
| Attack | Stop. 기존 Brain Combat Task의 사격 갱신 유지 |
| 나머지 상태·기억 없는 Chase | Stop. 별도 이동 정책 연결 전 안전 정지 |

착지 알림으로 BrainState 변경·발사 GA 재활성화·쿨다운 초기화 없음. 재진입 시 순찰 목적지·대기 시간 재선택, Chase 이동 재요청. 중단 지점의 정확한 진행률 복원 범위 제외

## Blueprint Task 생성

`StateTreeTaskBlueprintBase` 기반 `STT_NPC_GroundReady` 생성. Base의 **GroundState 부모**에 배치. Idle·Patrol·ChaseWalk 각각에 중복 배치 금지

- EntityContext 입력에 기존 Base Context 바인딩. `StateEventBus` 참조 사용
- 별도 발행 여부 Bool 불필요. 첫 Tick 발행 후 Task 성공 완료
- Class Defaults의 **Should State Change on Reselect** 해제. 자식 전이 시 Ground 알림 재발행 방지
- **Considered for Completion** 해제. Ground 완료 여부에 영향 없는 지속 Task
- **Should Call Tick Only on Events** 해제. 첫 Tick 실행 확보

EnterState: 실행 없음. Running 유지, Finish Task 호출 없음

Tick: `Publish Locomotion State(StateEvent = GroundedEntered, Source = Self, Target = EntityContext.StateEventBus) → Finish Task(true)`. 완료된 Task의 후속 Tick 없음

초기 Base StartLogic이 NPC의 이벤트 구독보다 먼저 실행되는 순서 때문에 EnterState 대신 첫 Tick에서 발행. 의도 판별을 Tick 전이 조건으로 반복 구현하지 않고 Ground 진입 사실 한 번만 게시

Context의 StateEventBus 누락 시 `Print String` 오류 메시지와 `Finish Task(false)`. 무한 재시도·조용한 Valid 실패 처리 금지. 오류 원인은 Context 바인딩·초기화 경로에서 수정

ExitState: 추가 처리 없음. 첫 Tick 전 이탈 시 미발행 상태로 종료. Controller의 BrainState·타겟 기억 변경 없음

## Base 전이 수정

1. Falling의 접지 전이 목적지를 **GroundState**로 지정. Ground의 기본 자식 Idle 선택. Patrol 직접 지정 제거
2. GroundState 부모에 `On Event: StateTree.Event.LocomotionCommand` 전이 추가
3. Payload 타입 `FLocomotionStateCommandPayload` 지정. 각 조건에 이벤트 Payload.Command 바인딩

| Command 조건 | 목적지 |
|---|---|
| Patrol | GroundState/Patrol |
| Chase | GroundState/ChaseWalk |
| Stop | GroundState/Idle |

부모의 세 전이보다 우선하는 자식의 동일 명령 전이 제거. Patrol → Chase도 부모 전이 경유. ChaseWalk의 CancelChase → Idle, Patrol의 CancelPatrol → Idle 유지 가능

Ground → Falling의 공중 조건 및 Death 전이 유지. CancelSprint를 Ground 전체 → Idle로 처리하지 않음. Sprint 상태에서만 취소 처리

Patrol Task Exit의 CancelIdlePatrol, ChaseWalk Task Exit의 CancelChaseMove 유지. Falling 진입 전에 이전 이동 요청 정리. Idle 진입에서 이전 이동까지 별도 초기화하지 않음

## 검증 항목

- 지상 시작: 첫 Ground Tick 후 순찰 시작. 자식 전이마다 Ground 알림 반복 없음
- 공중 시작: Falling → Ground → 현재 의도에 따른 Patrol·Chase·Idle 선택
- Chase 도중 낙하: 이동 취소, 타겟 기억 유지, 착지 후 추적 재요청
- 공중에서 타겟·BrainState 변경: 착지 시 최신 의도 적용
- Attack 의도 착지: Base Idle, 기존 Brain Combat Task 유지. 착지 알림으로 추가 발사 요청 없음
- 순찰 비활성화: Base Idle 유지
- 사망·비권한 Pawn: 착지 의도 복구 없음
- Upper ReadyEntered 반복: Patrol·Chase 강제 Idle 전이 없음

## 전체 실행 순서

```text
Ground 최초 진입 / Falling 후 착지
└─ GroundState 선택 → 기본 자식 Idle
   └─ STT_NPC_GroundReady Enter → Running 유지
      └─ 첫 Tick → EventBus 참조 확인
         ├─ EventBus 누락 → 오류 출력 → Finish Task(false) → 연결 수정
         └─ GroundedEntered 게시 → Finish Task(true) → 후속 Tick 없음
            └─ Mediator → NPCCharacter → Controller
               ├─ 사망·공중·비권한 → 복구 요청 없음
               ├─ Controller·Mediator 누락 → 진단 출력 → 구성 수정
               └─ 현재 BrainState로 명령 발신
                  ├─ Idle·순찰 사용 → Patrol → RequestIdlePatrol
                  ├─ Chase·기억 있음 → ChaseWalk → RequestChaseMove
                  └─ Attack·기타 → Idle

Ground 자식 전이
└─ GroundReady 재진입 없음 → 알림 반복 없음

중단·낙하·사망
├─ Patrol Exit → CancelIdlePatrol
├─ ChaseWalk Exit → CancelChaseMove
└─ GroundReady Exit → 첫 Tick 전 이탈 시 알림 미발행
   └─ 다음 Ground 진입에서 새 알림 예약
```
