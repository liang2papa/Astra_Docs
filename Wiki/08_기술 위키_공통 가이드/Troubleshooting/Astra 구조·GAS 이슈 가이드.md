# Astra 구조·GAS 이슈 가이드

## 설계 의도

일정의 상태 추적과 기술 상세 분리

## 기록 범위

- 기존 남은 이슈·해결된 이슈의 기술 내용 이관
- 미해결 항목은 해결법 확정 문서가 아닌 현재 상태·검증 절차
- 현재 확인일: 2026-10-08
- 이관일: 2026-09-03
- 이번에 확인한 UI 대화·일부 C++ 이외의 전체 구현·PIE 재검증은 미수행
- 원인·해결 방법이 기록되지 않은 부분은 임의 보완하지 않음

## 남은 작업

### ARCH-001. EntityStatusModel 제거

- 상태: C++ 반영. StatusModel 참조 제거 확인

#### 현재 기록·검증 범위

- 능력치를 Entity ASC의 AttributeSet으로 이전. 현재 소스에 StatusModel 참조 없음
- Player·NPC 공통 Entity 초기화 경로 사용. SoulEnergy는 PlayerState ASC에 분리
- 상세 경계는 [시스템 아키텍처](../../03_시스템%20설계/시스템%20아키텍처.md) 참고

### ARCH-005. WeaponComponent 클래스 분할

- 상태: C++ 기능 분리 반영

#### 현재 기록·검증 범위

- WeaponComponent는 외부 Facade로 유지한다.
- Aim·Carry·Reload·Recoil·AttackSequence·AttackCommit Handler를 내부 소유한다.
- 공통 참조는 FWeaponHandlerContext에서 관리한다.
- 외부 객체는 Handler를 직접 참조하지 않는다.
- Handler 간 직접 참조를 만들지 않는다.
- PhysicalProjectile 생성과 탄약 소비는 AttackCommitHandler가 담당한다.
- Handler는 WeaponComponent를 Outer로 생성하고 `UPROPERTY(Transient)` 참조로 수명을 유지한다.
- 공통 Context는 참조만 보관하고 Handler 간 직접 참조를 만들지 않는다.
- ASC·Inventory·WeaponInstance 교체 시 Context를 갱신한다.
- 무기 Instance 해제 전에 진행 행동 취소와 Delegate 해제를 수행한다.
- Component 생성자에서 초기화 전 외부 소유 Mesh를 역참조하지 않는다.
- Muzzle·Hand·Holster Socket 누락은 원점 Transform으로 대체하지 않고 실패로 처리한다.
- [WeaponComponent 기능 분리](../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/WeaponComponent%20%EA%B8%B0%EB%8A%A5%20%EB%B6%84%EB%A6%AC.md)

### ARCH-006. EntityLocomotionComponent 클래스 분할

- 상태: C++ 책임 분리 반영. 에셋 연결·동작별 회귀 확인 별도

#### 현재 기록·검증 범위

- Entity 공통 이동·행동 경계 분리. 현재 NPC도 공통 LocomotionComponent 사용
- Blueprint 행동 Task는 명령 연결 담당. NPC AIController는 추적·순찰 PathFollowing 실행 담당

### ARCH-007. PlayerContext 재생성 경고 제거

- 상태: 현재 C++에서 경고·재생성 경로 제거 확인. 실행 중 재발 여부 확인 별도

#### 증상

`BP_PlayerCharacter_C_0: PlayerContext was missing and has been recreated.`

#### 현재 기록·검증 범위

- 경고를 정상 초기화로 간주하지 않는다.
- PlayerContext 생성 전에 MovementComponent 등 필수 참조가 준비되는 시점을 확인한다.
- 초기화 순서를 고치거나 생성 시점을 늦추는 방식을 비교한다.
- 2026-10-08 소스 확인: 해당 경고 문구와 StatusModel 참조 없음
- 과거 증상 기록 유지. 전체 초기화 회귀 완료를 의미하지 않음

### ARCH-002. AbilitySet + 입력 태그 기반 부여 구조

- 상태: 이관 대기

#### 현재 기록·검증 범위

- PlayerCharacter의 기능별 Ability 직접 참조를 줄이고 AbilitySet과 GameplayTag 기반 활성화로 이관한다.

### ARCH-003. NPC용 ASC 소유 위치 확정

- 상태: NPC Pawn 소유 확정·C++ 반영

#### 현재 기록·검증 범위

- EntityCharacter가 자신의 ASC를 소유. Owner·Avatar 모두 해당 Character로 초기화
- HP·Effect·쿨다운은 육체 귀속. Player 고유 SoulEnergy는 PlayerState ASC 소유
- 영혼 결속 Possess/해제 실제 연결은 후속 작업

### ARCH-004. 영혼 결속 Ability 구현

- 상태: 미구현

#### 현재 기록·검증 범위

- `Possession`, `DirectSkill`, `Transform`, `Totem`은 설계만 정리된 상태다.
- 플레이어 무기 어빌리티와 아스트라 어빌리티 셋이 섞이지 않도록 태그와 부여/회수 규칙을 구현해야 한다.

## 해결 기록


## 상태 목록·관련 문서

- [남은 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%EB%82%A8%EC%9D%80%20%EC%9D%B4%EC%8A%88.md)
- [해결된 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%ED%95%B4%EA%B2%B0%EB%90%9C%20%EC%9D%B4%EC%8A%88.md)
- [Troubleshooting 목록](Troubleshooting%20%EB%AA%A9%EB%A1%9D.md)
- [정리 근거](../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
