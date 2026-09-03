# Astra 구조·GAS 이슈 가이드

## 설계 의도

일정의 상태 추적과 기술 상세 분리

## 기록 범위

- 기존 남은 이슈·해결된 이슈의 기술 내용 이관
- 미해결 항목은 해결법 확정 문서가 아닌 현재 상태·검증 절차
- 기존 기록 기준일: 남은 이슈 2026-09-02 / 해결 이슈 2026-08-28
- 이관일: 2026-09-03
- 이번에 확인한 UI 대화·일부 C++ 이외의 전체 구현·PIE 재검증은 미수행
- 원인·해결 방법이 기록되지 않은 부분은 임의 보완하지 않음

## 남은 작업

### ARCH-001. EntityStatusModel 단계적 이전

- 상태: 단계적 이전 대기

#### 현재 기록·검증 범위

- 상태·스탯 중 GAS Attribute/Tag가 소유해야 할 항목과 StateTree 입력 컨텍스트가 소유할 항목을 분리한다.
- NPC도 동일한 ASC/AttributeSet 기반을 사용할 수 있도록 PlayerState 전용 소유로 고정하지 않는다.

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
