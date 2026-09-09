# Astra 구조·GAS 이슈 가이드

## 설계 의도

일정의 상태 추적과 기술 상세 분리

## 기록 범위

- 기존 남은 이슈·해결된 이슈의 기술 내용 이관
- 미해결 항목은 해결법 확정 문서가 아닌 현재 상태·검증 절차
- 현재 확인일: 2026-09-09
- 이관일: 2026-09-03
- 이번에 확인한 UI 대화·일부 C++ 이외의 전체 구현·PIE 재검증은 미수행
- 원인·해결 방법이 기록되지 않은 부분은 임의 보완하지 않음

## 남은 작업

### ARCH-001. EntityStatusModel 제거

- 상태: 논의 필요

#### 현재 기록·검증 범위

- 능력치를 GAS AttributeSet으로 이전하고 EntityStatusModel을 제거한다.
- AttributeSet의 조회·변경 경로를 먼저 확정한다.
- NPC도 같은 ASC·AttributeSet 구조를 사용하도록 소유 위치와 접근 경계를 함께 결정한다.

### ARCH-005. WeaponComponent 클래스 분할

- 상태: 논의 필요

#### 현재 기록·검증 범위

- NPC 제작 전에 수행한다.
- WeaponComponent가 PlayerCharacter에서 받는 이벤트와 GAS가 사용하는 API를 먼저 목록화한다.
- 동작별 Handler를 WeaponComponent가 소유하는 안을 우선 검토한다.
- Handler의 결과가 외부 전달을 요구하면 WeaponComponent가 Character로 반환한다.
- 클래스 자체 분할안과 Handler 소유안을 비교한 뒤 책임 경계를 확정한다.

### ARCH-006. EntityLocomotionComponent 클래스 분할

- 상태: 설계 필요

#### 현재 기록·검증 범위

- NPC 제작 전에 수행한다.
- 현재 GAS가 사용하는 기능과 Entity 공통 기능을 기준으로 책임을 분리한다.
- 세부 Handler와 클래스 경계는 미정이다.

### ARCH-007. PlayerContext 재생성 경고 제거

- 상태: 원인 분석 필요

#### 증상

`BP_PlayerCharacter_C_0: PlayerContext was missing and has been recreated.`

#### 현재 기록·검증 범위

- 경고를 정상 초기화로 간주하지 않는다.
- PlayerContext 생성 전에 MovementComponent 등 필수 참조가 준비되는 시점을 확인한다.
- 초기화 순서를 고치거나 생성 시점을 늦추는 방식을 비교한다.
- 현재 경고 출력과 런타임 재생성 경로는 `PlayerCharacter.cpp`에 남아 있다.

### ARCH-002. AbilitySet + 입력 태그 기반 부여 구조

- 상태: 이관 대기

#### 현재 기록·검증 범위

- PlayerCharacter의 기능별 Ability 직접 참조를 줄이고 AbilitySet과 GameplayTag 기반 활성화로 이관한다.

### ARCH-003. NPC용 ASC 소유 위치 확정

- 상태: 설계 확정 대기

#### 현재 기록·검증 범위

- 빙의·변신·일반 NPC 모두 재사용할 수 있도록 Pawn/PlayerState 중 실제 싱글플레이 요구에 맞는 소유 위치를 확정한다.

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
