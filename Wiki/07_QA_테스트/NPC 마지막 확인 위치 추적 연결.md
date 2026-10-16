# NPC 마지막 확인 위치 추적 연결

2026-10-08 C++ 반영. 빌드 사용자 수행. UHT·Blueprint Compile·StateTree Compile·PIE 결과 미검증

## 동작과 설정

- Sight 감지 및 LineOfSightTo 통과 시에만 마지막 확인 위치 갱신
- 벽 가림 또는 타겟 Actor 소멸 시 공격·현재 Actor Focus·기존 이동 중지 및 위치 기억 유지
- `LostTargetMoveDelay` 경과 후 기억 위치로 접근. 기본값 0은 즉시 출발. 대기 시간은 DA에서 지정
- 목적지가 NavMesh 밖 또는 다른 섬에 있는 경우 부분 경로 탐색으로 도달 가능한 끝점 산출. 끝점까지의 실제 이동은 완전 경로로 요청. 세계 전체의 수학적 최근접 지점 보장은 제외
- 접근 지점 도착 후 **기존 `GiveUpTime`** 동안 대기·재감지. 별도 배회 이동 추가 없음
- 재발견 시 상실·탐색 대기와 이전 요청 해제 후 가시 타겟 추적·공격 재개
- 경로 부재 시 `ChaseRetryDelay` 간격으로 `ChaseMaxRetries`만큼 추가 재시도. 재시도 소진 후 가려진 타겟은 현재 위치에서 GiveUpTime 대기 후 기억 제거
- `GiveUpTime` 시작 기준을 기존 타겟 상실 시점에서 마지막 위치 접근 완료 시점 또는 경로 실패 복구 진입 시점으로 변경. 중복 SearchTime 필드 없음
- `ChaseAttackRangeFraction`: 기존 BP의 사거리 × 0.2를 설정으로 이전한 값. 가시 타겟 도착 반경은 `min(MoveAcceptanceRadius, EffectiveRange × ChaseAttackRangeFraction)`. 기억 위치 접근에는 MoveAcceptanceRadius 사용
- 재장전 설정과 GA 변경 없음

DA_NPC_Humanoid → Combat Tendency에서 기존 GiveUpTime·MoveAcceptanceRadius와 새 LostTargetMoveDelay 확인. 재시도 설정 기본값 0.5초/3회

## 1. NPCBrain Root

`ST_AIBrain` Root에 **NPC Update Target Tracking** Task 추가

- AIController에 두뇌 Context의 AIController 바인딩
- 완료 판정에서 제외. Task의 Running 유지
- Idle·Chase·Attack·Dead를 포함한 상위 Root에서 지속 활성화
- Controller Tick 코드 및 BP Timer 중복 추가 금지

해당 Task 누락 시 공격·이동 종료 후 기억 만료 갱신 불가

## 2. 두뇌 전이

타겟 Actor 소멸 후에도 위치 기억 유지 필요. 기존 `NPC Has Current Target`을 아래 기억 조건으로 교체

| 전이 | 조건 | Invert |
|---|---|---|
| Idle → Attack | 타겟이 사정거리 안에 있는가 | false |
| Idle → Chase | NPC Has Target Memory | false |
| Chase → Attack | 타겟이 사정거리 안에 있는가 | false |
| Chase → Idle | NPC Has Target Memory | true |
| Attack → Chase | NPC Has Target Memory AND 사거리 Condition | false / true |
| Attack → Idle | NPC Has Target Memory | true |

- 모든 AIController를 동일 Context로 바인딩
- Idle의 Attack 후보를 Chase보다 우선 평가
- C++ 사거리 Condition에 가시성 검사 추가. 벽 가림 시 범위 안에서도 일반 결과 false, Invert 결과 true 반환
- 독립적인 시야 조건 필요 시 새 **NPC Has Visible Target** 사용 가능
- 기존 NPC Combat Attack·사망 전이·상태 설정 Task 유지
- 두뇌 Chase에 상실 후 출발 전 대기·마지막 위치 이동·도착 후 대기 포함. Controller의 GetTargetTrackingPhase로 세부 단계 확인. 새 Brain enum 상태 추가 없음

## 3. 기존 STT_NPC_ChaseWalk 변경

기존 Blueprint Task 유지. 기존 AI MoveTo·AcceptanceRadius 곱셈·Exit의 StopMovement 제거. 새 C++ API와 기존 BP AI MoveTo 동시 실행 시 이동 요청 간 취소 가능성 존재

부모가 제공하는 NPC Pawn/EntityCharacter 참조와 Controller Cast 유지. 네이티브 AstraNPCAIController로 Cast 가능

Task 변수 `NPCController`를 AstraNPCAIController Object Reference로 추가. 매 EnterState에 해당 NPC의 Controller 확보

### EnterState

1. EnterState → 부모가 바인딩한 Pawn 유효성 검사.
2. Pawn.GetController → Cast to AstraNPCAIController → 변수 NPCController에 저장.
3. NPCController.RequestChaseMove 호출. Target 핀은 저장한 NPCController.
4. Task의 Running 유지. 반환값 Reached/Waiting/Moving에 대한 FinishTask 호출 금지
5. Pawn Invalid/Cast Failed는 FinishTask(Succeeded=false).

목적지 선택·도착 알림·재시도 횟수는 Controller에서 관리. Task의 LastKnownLocation 별도 저장 및 매 Tick RequestChaseMove 호출 금지. Enter에서 RequestChaseMove 1회 호출

### Tick

1. Event Tick → IsValid(NPCController).
   - Invalid → FinishTask(false).
2. NPCController.UpdateChaseMove 호출.
3. 반환 상태를 디버그 표시용으로 사용. 정상 참조 유지 중 Running 유지
   - Waiting: 출발 전 대기, 도착 후 탐색, 재요청 간격 대기.
   - Moving: 이동 중.
   - Reached: Controller가 이미 도착을 처리하고 GiveUpTime을 시작한 상태.
   - Failed: 제한된 재시도 소진 또는 의존성 문제. 가려진 타겟은 Controller의 탐색 만료 대기. 가시 타겟은 목적지 변경 시 재시도 허용
   - Idle: 재발견·기억 만료·취소 직후 가능한 상태. 두뇌 전이 대기

**도착 직후 FinishTask(true) 호출 금지.** 같은 두뇌 Chase에서 재발견 시 이동 재개를 위해 몸의 Chase 유지. Controller에서 도착 처리 및 탐색 진입 수행

기억 만료 시 두뇌 Idle 전이 → ChaseCompleted 발신 → 몸의 Chase 종료

### ExitState

Event ExitState → IsValid(NPCController) → CancelChaseMove(Target=NPCController) → 변수 None으로 정리.
CancelChaseMove에서 이동 요청만 취소. 기억·탐색 시간 유지. 다음 상태 Task에서 해당 행동 시작

## 4. Base 트리

- Chase 명령 → Base Chase, CancelChase → Base Idle 연결 유지
- STT_NPC_ChaseWalk를 완료 판정에 포함. Running 중 Chase 유지. 일회성·Debug Task는 완료 판정에서 제외
- Root의 무조건적인 CancelSprint → Idle 전이 시 Upper Ready/Fire/Reload 알림에 의한 Chase 중단 가능성 존재. CancelSprint를 Sprint 종료가 필요한 상태에만 적용
- Controller Cast 실패 등 Task 자체 실패에 대한 Base 복구 전이 별도 설정. Controller의 경로 재시도 상태를 BP에서 중복 처리 금지

## 5. PIE 확인 항목

1. 가시 타겟 이동: LastKnownTargetLocation 갱신 및 추적.
2. 벽 뒤로 이동: 공격 중지, 기억 좌표 고정, LostTargetMoveDelay 동안 정지.
3. 대기 종료: Actor의 숨은 현재 위치가 아닌 기억 좌표로 이동.
4. Actor Destroy: null 접근 없이 같은 기억 이동 수행.
5. NavMesh 밖/다른 섬: 부분 경로 끝점 접근 후 GiveUpTime 대기.
6. 도착 후 GiveUpTime 만료: 기억 제거, 두뇌 Idle, Base Idle.
7. 출발 전/이동 중/도착 후 재발견: 이전 요청 정리, 살아 있는 타겟 위치로 갱신, 추적/공격 재개.
8. 경로 실패: 제한 재시도 후 대기/포기. 매 프레임 MoveTo 재발신 없음.
9. 사망/UnPossess/PIE 종료: 이동·공격·타겟 Destroy 구독·기억 정리.

디버그 출력용 API: GetTargetTrackingPhase·HasVisibleTarget·HasTargetMemory·GetLastKnownTargetLocation·GetChaseMoveStatus·GetChaseDestination

## 전체 실행 순서

```text
NPCBrain Root → NPC Update Target Tracking (지속)
 → Sight + LineOfSightTo 통과 → 현재 위치 기억 → Visible
    ├─ 사거리 안 → Attack → 방향 정렬 → GA 발사
    └─ 사거리 밖 → Chase → Base ChaseWalk Enter → RequestChaseMove

벽 가림 / Actor 소멸
 → 공격·Actor Focus·기존 이동 취소 → 마지막 위치 유지 → LostDelay
 → LostTargetMoveDelay 종료 → LastKnownMove
 → ChaseWalk Tick.UpdateChaseMove
 → 경로 탐색(부분 경로 허용) → 도달 가능한 끝점 → MoveTo
    ├─ 도착 → Searching → 기존 GiveUpTime 시작
    │          → Task는 Running 유지
    │          ├─ 재발견 → 이전 요청 정리 → Visible → Chase/Attack
    │          └─ 시간 만료 → 기억 제거 → 두뇌 Idle
    │                         → ChaseCompleted → Base Idle
    └─ 실패 → ChaseRetryDelay 대기 → 제한 재시도
               → 소진 + 가려짐 → 현재 위치에서 GiveUpTime → Idle

ChaseWalk Exit (정상 전이 / 실패 복구 / 사망 / 취소)
 → CancelChaseMove → 요청 ID 무효화 → StopMovement
 → 다음 상태의 Task 실행

UnPossess / 사망 / Controller 종료
 → ForgetCurrentTarget → 공격·이동·Focus·Destroy 구독·기억 정리
```
