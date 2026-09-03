# 수량 입력과 Stack 분할

## 설계 의도

수량 입력 위젯과 인벤토리 실행 책임 분리

## 위젯 구조

```text
WBP_SplitStackDialog
└─ SizeBox_Root
   └─ Border_BG
      └─ VerticalBox_Root
         ├─ Text_ItemName
         ├─ HorizontalBox_Quantity
         │  ├─ Text_QuantityLabel
         │  └─ SpinBox_Quantity
         └─ HorizontalBox_Buttons
            ├─ Button_Confirm
            └─ Button_Cancel
```

## 데이터

- Dialog 멤버: MaximumSplitQuantity (Integer)
- Confirm Dispatcher: OnConfirmRequested(SplitQuantity : Integer)
- Cancel Dispatcher: OnCancelRequested()
- 상위 Menu 멤버: ActiveSplitInventorySlot, 기본값 -1
- Dialog는 InventorySlot·InventoryComponent 직접 참조 없음

SpinBox: Min Value 1 / Delta 1 / 소수 자릿수 Min·Max 0

최대값: 현재 Stack Quantity - 1. Component가 전달한 값 사용

## 해결된 수량 1 오류

- 증상: 다른 수량을 입력해도 실제 분할은 1개
- 원인: SetDialogData가 SpinBox Max만 설정. MaximumSplitQuantity 멤버 저장 누락
- 해결: InMaximumSplitQuantity → Set MaximumSplitQuantity
- 확인: 2026-09-03 대화에서 사용자 해결 확인

`Set Max Value`는 SpinBox 범위 변경. 별도 Blueprint 변수를 변경하지 않음

앞선 입력 확정 시점·Dispatcher 연결 가설은 이 사례의 확정 원인 아님

## 전체 노드 실행 순서

```text
ReceiveMenuInitialized
→ Bind OnConfirmRequested → HandleSplitConfirmed
→ Bind OnCancelRequested → HandleSplitCancelled

나누기 요청
→ OpenSplitStackDialog(InventorySlot)
  ├─ 분할 불가 → 실패 안내 또는 요청 거부 → 종료
  └─ 가능 → DisplaySplitStackDialog(InventorySlot, ItemInstance, Quantity-1)
→ Set ActiveSplitInventorySlot = InventorySlot
→ Context Menu / 장착 선택 Menu Collapsed
→ SetDialogData(ItemInstance, InMaximumSplitQuantity)
  → Item 이름 Set Text
  → Set MaximumSplitQuantity = InMaximumSplitQuantity
  → SpinBox.Set Min Value(1)
  → Set Max Value(InMaximumSplitQuantity)
  → Set Min Slider Value(1)
  → Set Max Slider Value(InMaximumSplitQuantity)
  → Set Value(1)
→ Dialog Visible → Layout Prepass → 화면 경계 위치 보정

Button_Confirm.OnClicked
→ SpinBox.GetValue → Round → Clamp(1, MaximumSplitQuantity)
→ Call OnConfirmRequested(SplitQuantity=Clamp 결과)
→ HandleSplitConfirmed(SplitQuantity)
→ Get Menu Component
→ RequestSplitStack(ActiveSplitInventorySlot, SplitQuantity)
  ├─ 성공 → 슬롯 갱신 → CloseTransientPanels
  ├─ 일반 실패 → DisplayActionFailure → Dialog 유지
  └─ 메뉴 닫힘·연결 부재 → false, 데이터 변경 없음

Button_Cancel.OnClicked
→ Call OnCancelRequested
→ HandleSplitCancelled
→ Dialog Collapsed → ActiveSplitInventorySlot = -1

CloseTransientPanels 또는 전체 메뉴 닫기
→ Dialog Collapsed → ActiveSplitInventorySlot = -1
```

## 검증

- 정상 입력: 30개 Stack에서 10개 선택 → 20 + 10
- 최소·최대: 1 / Quantity-1
- 실행 전 슬롯 상태 변경·빈 슬롯 부족: Component 실패 처리 확인
- 일반 분할 완료와 실패·취소 회귀 검증을 구분


## 근거와 관련 문서

- 대화: 장비와 인벤토리 구현 방향 확인 — SetDialogData 확인 후 사용자 해결 확인
- [대화·문서·코드 확인 범위](../../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
- [팝업 위치와 화면 경계](%ED%8C%9D%EC%97%85%20%EC%9C%84%EC%B9%98%EC%99%80%20%ED%99%94%EB%A9%B4%20%EA%B2%BD%EA%B3%84.md)
- [초기화 변수와 이벤트 전달](%EB%8F%99%EC%A0%81%20%EC%9C%84%EC%A0%AF%20%EC%83%9D%EC%84%B1%EA%B3%BC%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EA%B0%B1%EC%8B%A0.md)
- [Unreal 지식 목록](../Unreal%20%EC%A7%80%EC%8B%9D%20%EB%AA%A9%EB%A1%9D.md)
- [99_작업 규칙](../../../99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99/99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99.md)
