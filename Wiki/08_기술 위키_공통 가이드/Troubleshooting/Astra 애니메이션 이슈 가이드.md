# Astra 애니메이션 이슈 가이드

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

### ANIM-004. Draw/Holster와 Pistol Idle 포즈 호환성

- 상태: 임시 완화 · 애셋 교체 대기

#### 현재 기록·검증 범위

- Blend In/Out `0.2초 + Exponential In`으로 임시 완화했다.
- 호환 애니메이션 세트를 확보한 뒤 다시 조정해야 한다.

### ANIM-005. 권총 Equip 시 손가락 관통

- 상태: 보정 대기

#### 현재 기록·검증 범위

- 양손 그립은 대체로 맞지만 오른손 손가락 일부가 왼손을 관통한다.
- 애니메이션 교체 또는 Control Rig의 손가락 보정이 필요하다.

### ANIM-006. 무기 종류별 Animation Layer 확장

- 상태: 확장 대기

#### 현재 기록·검증 범위

- 현재는 권총 기준 `ABP_Unarmed`에 직접 구성되어 있다.
- Rifle 등으로 확장할 때 Weapon Animation Layer Interface와 무기별 Linked Anim Layer 전환을 적용한다.

## 해결 기록

### ANIM-001. 상하 조준 시 허리 굽힘 부족과 왼팔 머리 관통

- 상태: 해결 완료
- 기존 현상: 상하 조준에서 척추 굽힘이 부족하고 왼팔이 머리 또는 몸통을 관통했다.
- 완료 기준: 허용된 카메라 Pitch 범위에서 조준 자세와 총구 방향이 정상 유지되는 것을 확인했다.
- 완료 확인: 2026-08-28 사용자 확인

### ANIM-002. Pistol Idle과 Fire 포즈 불일치 및 Idle 루프 끊김

- 상태: 해결 완료
- 기존 현상: Idle 반복 경계와 Idle↔Fire 전환에서 팔·어깨·총기가 튀었다.
- 완료 기준: Idle 루프와 Fire 전환이 눈에 띄는 단절 없이 재생되는 것을 확인했다.
- 완료 확인: 2026-08-28 사용자 확인

### ANIM-003. AimOffset Pitch 증가에 따른 왼손 이탈

- 상태: 해결 완료
- 원인: 완성된 무기 애니메이션 위에 FABRIK을 추가하여 기존 손·팔 관계를 다시 강제로 보정했다.
- 해결: 완성 애니메이션 경로의 중복 FABRIK을 제거했다.

### Fire Montage의 고정 팔·어깨 포즈

- 상태: 해결 완료
- 원인: AimOffset 뒤의 Montage Slot이 조준 Pose를 덮는 경로
- 해결: UpperBody Slot → AimOffset 순서로 수정
- [Pose 합성 순서 가이드](../Unreal/Animation/Slot%C2%B7AimOffset%C2%B7IK%20%EC%A0%81%EC%9A%A9%20%EC%88%9C%EC%84%9C.md)
## 상태 목록·관련 문서

- [남은 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%EB%82%A8%EC%9D%80%20%EC%9D%B4%EC%8A%88.md)
- [해결된 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%ED%95%B4%EA%B2%B0%EB%90%9C%20%EC%9D%B4%EC%8A%88.md)
- [Troubleshooting 목록](Troubleshooting%20%EB%AA%A9%EB%A1%9D.md)
- [정리 근거](../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
