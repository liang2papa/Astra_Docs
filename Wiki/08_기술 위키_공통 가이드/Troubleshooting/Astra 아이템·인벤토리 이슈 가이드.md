# Astra 아이템·인벤토리 이슈 가이드

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

### ITEM-002. WorldItem 상호작용 방식 전환과 Soft Reference 로딩

- 상태: 상호작용·로딩 후속 작업

#### 현재 기록·검증 범위

- `AWorldItemActor`는 여러 ItemInstance를 보관하고 수용 가능한 수량만 Inventory·Gear로 이전한다.
- 현재 Overlap 즉시 획득을 향후 별도 Interaction 입력 방식으로 교체한다.
- ItemDataAsset의 Soft ItemStaticMesh 자동 로딩과 취소 정책은 아직 연결하지 않았다.

### ITEM-003. Bag Capacity 감소 Transaction

- 상태: 미구현

#### 현재 기록·검증 범위

- Bag은 일반 Auto Equip과 Inventory↔Gear 이동에서 현재 제외된다.
- Compact 후 새 Capacity 밖 Item과 해제 Bag을 하나의 WorldItemActor로 옮기고, 실패 시 원상 복구하는 Transaction을 구현한다.

### ITEM-004. SaveData와 레벨 전환 복원

- 상태: 미구현

#### 현재 기록·검증 범위

- ItemInstance UObject와 Runtime Item ID를 직접 저장하지 않고 별도 SaveData Schema를 구현한다.
- 새 World의 ItemInstanceFactorySubsystem을 통해 Instance와 ID를 재발급한 뒤 Inventory·Gear를 복원한다.
- Preferred Ammo Map도 현재 Transient이므로 Player SaveData에 포함해야 한다.

### ITEM-005. Inventory·Gear UI 후속 구현

- 상태: Stack 분할 완료 · Tooltip C++ 반영 · Widget 연결 진행

#### 현재 기록·검증 범위

- PlayerController 소유 UInventoryMenuComponent·UInventoryMenuWidget, Toggle Input과 Equip·Unequip·Split·Drop 요청 C++ API 구현
- 7×7 Inventory/Gear Slot·아이콘·Inventory/Gear Context Menu 연결, 장비 해제 동작 확인
- 장착 선택 enum의 멤버 저장 누락 수정 안내 후 사용자 수정 확인. 전체 Equip 조합 회귀는 별도
- Stack 분할: SetDialogData 멤버 저장 누락 수정 후 2026-09-03 사용자 해결 확인
- Tooltip은 `FItemTooltipViewData` 값 스냅샷을 MenuComponent가 구성해 Widget Event에 전달한다.
- 현재 ViewData는 공통 Item·Equipment·Weapon 값을 포함한다. Ammunition 전용 필드와 중첩 Struct 분리는 후속 확장이다.
- Tooltip 공통값·종류별 영역·희귀도 색상·아이콘 연결을 완성한다.
- Drop 결과·실패 메시지·Locked Capacity·메뉴 열기/닫기 표현을 완성한다.
- Preferred/Reserved 탄약: Stack 독립 설계 유지. 대표 Stack·아이콘·수량 표현 확정 잔여
- [Stack 분할 해결 상세](../Unreal/UMG/%EC%88%98%EB%9F%89%20%EC%9E%85%EB%A0%A5%EA%B3%BC%20Stack%20%EB%B6%84%ED%95%A0.md)
- [장착 옵션과 초기화 값 전달](../Unreal/UMG/%EB%8F%99%EC%A0%81%20%EC%9C%84%EC%A0%AF%20%EC%83%9D%EC%84%B1%EA%B3%BC%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EA%B0%B1%EC%8B%A0.md)
- [Tooltip ViewData와 ViewModel](../Unreal/UMG/Tooltip%20ViewData%EC%99%80%20ViewModel.md)

### ITEM-006. 탄약이 0인 무기 장착 시 자동 장전

- 상태: 미구현

#### 현재 기록·검증 범위

- Inventory에서 무기 장착이 확정된 뒤 장전 탄약이 0인지 확인한다.
- 0이면 직접 탄약을 변경하지 않고 기존 Reload 요청 흐름으로 전달한다.
- 발사 요청 시 빈 탄창 자동 재장전과 별개의 장착 편의 기능이다.

### ITEM-007. WeaponInstance의 장전 탄약 소유 구조 변경

- 상태: 논의 필요

#### 현재 기록·검증 범위

- 버리기와 재획득을 거쳐도 장전 탄약을 WeaponInstance 상태로 유지한다.
- Inventory의 탄약 예약 구조는 현재 방식으로 유지한다.
- 두 상태의 동기화·해제 시점과 Drop 직렬화 규칙은 미정이다.
- 인벤토리에서 즉시 버리기는 우선 사용하지 않는다.

### ITEM-008. 무기 장착 슬롯 선택 방식 변경

- 상태: 미구현

#### 현재 기록·검증 범위

- 두 슬롯이 비었으면 Weapon1에 즉시 장착한다.
- Weapon1만 비었으면 Weapon1에 즉시 장착한다.
- Weapon2만 비었으면 Weapon2에 즉시 장착한다.
- 두 슬롯이 모두 찼을 때만 선택 옵션을 표시한다.

## 해결 기록

### Stack 분할 수량 1 고정

- 상태: 해결 완료
- 원인: SpinBox Max 설정과 MaximumSplitQuantity 멤버 저장 혼동
- 해결: InMaximumSplitQuantity → Set MaximumSplitQuantity
- 완료 근거: 장비와 인벤토리 구현 방향 확인 — 2026-09-03 사용자 해결 확인
- [정상·실패·취소 전체 연결](../Unreal/UMG/%EC%88%98%EB%9F%89%20%EC%9E%85%EB%A0%A5%EA%B3%BC%20Stack%20%EB%B6%84%ED%95%A0.md)

### 장착 옵션의 기본 enum 전달

- 상태: 수정 반영 확인 · 전체 장착 조합 회귀 별도
- 원인: InitializeOption 입력을 텍스트에만 사용하고 EquipmentSlot 멤버에 저장하지 않음
- 해결: InEquipmentSlot → Set EquipmentSlot → 표시 갱신
- [동적 옵션 초기화](../Unreal/UMG/%EB%8F%99%EC%A0%81%20%EC%9C%84%EC%A0%AF%20%EC%83%9D%EC%84%B1%EA%B3%BC%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EA%B0%B1%EC%8B%A0.md)
## 상태 목록·관련 문서

- [남은 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%EB%82%A8%EC%9D%80%20%EC%9D%B4%EC%8A%88.md)
- [해결된 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%ED%95%B4%EA%B2%B0%EB%90%9C%20%EC%9D%B4%EC%8A%88.md)
- [Troubleshooting 목록](Troubleshooting%20%EB%AA%A9%EB%A1%9D.md)
- [정리 근거](../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
