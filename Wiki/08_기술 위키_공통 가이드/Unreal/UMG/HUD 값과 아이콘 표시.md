# HUD 값과 아이콘 표시

```text
게임 상태 변경 → Controller / UI Component 중계
→ Widget 표시 이벤트
→ ProgressBar / Text / Image 갱신
```

## 설계 의도

UI는 게임 상태를 읽어 표시. 상태 변경은 Model·Component에서 처리

## 초기화와 갱신

- 이벤트 구독 후 현재 상태를 한 번 동기화
- 초기화 전 무기 없음과 장착 완료 후 무기 있음은 서로 다른 정상 시점
- 무기를 숨길 때는 마지막 숨김 명령 전달
- 메뉴·Pawn 교체 시 이전 대상 Delegate 정리

## Soft Texture와 빈 슬롯

- 아이템 아이콘 Soft Reference: Set Brush from Soft Texture
- 로드된 Texture: Set Brush from Texture
- Soft 경로 지정 여부와 실제 로드 완료 여부 구분
- 빈 슬롯의 ItemInstance = None은 정상 표시 상태
- 빈 슬롯으로 바뀌면 이전 아이콘·수량·참조도 초기화
- Equal (SoftObjectReference)의 B를 비워 비교하던 과거 방식은 해당 대화에서 컴파일 오류. 전용 Is Valid Soft Object Reference 사용

전용 검사는 Soft 참조가 null인지 확인하는 기능. 실제 로드 완료 검사와 구분. [Epic Blueprint API](https://dev.epicgames.com/documentation/unreal-engine/BlueprintAPI/Utilities/IsValidSoftObjectReference)

Soft Texture 설정 함수: [Epic UImage](https://dev.epicgames.com/documentation/unreal-engine/API/Runtime/UMG/UImage?lang=en-US)

## 전체 노드 실행 순서

```text
DisplayHealth(CurrentHealth, MaxHealth)
→ MaxHealth > 0
  ├─ True → CurrentHealth / MaxHealth → Clamp(0,1)
  │        → PB_Health.SetPercent
  │        → Round Current / Max → Format Text("{Current} / {Max}")
  │        → TXT_HealthValue.SetText
  └─ False → PB_Health.SetPercent(0)
           → TXT_HealthValue.SetText("0 / 0")

SetSlotData 또는 SetEquipmentInstance
→ 입력 Instance를 멤버에 저장
→ 빈 슬롯인가?
  ├─ 예 → Set Brush from Texture(None, Match Size=false)
  │      → 수량 텍스트·외곽선 상태 초기화
  └─ 아니오 → Get Item Data → ItemIcon
      → Is Valid Soft Object Reference
        ├─ True → Set Brush from Soft Texture(ItemIcon, Match Size=false)
        └─ False → Set Brush from Texture(None, Match Size=false)
      → 수량·잠금·선택 상태 표시

DisplayCrosshair(Texture, bVisible, CrosshairScale)
→ bVisible
  ├─ True → Image.SetBrushFromTexture(Texture, Match Size=false)
  │        → Set Render Scale(Vector2D(Scale, Scale))
  │        → Set Visibility(Not Hit-Testable)
  └─ False → Set Visibility(Collapsed)

메뉴·HUD 종료
→ 표시 종료 → 중계 계층 구독 해제
```

## Astra 확인 범위

- 기본 HUD: 체력·스태미나·탈진·무기 아이콘·현재/최대 탄약·크로스헤어 정상 진행 사용자 확인
- 현재 DisplayAmmo: CurrentAmmo / MaxLoadedAmmo / AvailableAmmo / bUsesAmmo의 4개 입력
- 기존 3개 입력 Blueprint 설명과 구분
- Available·Preferred·Reserved 표현, 해상도 전체 회귀는 별도 확인
- 근접무기도 무기별 크로스헤어 대상. 과거 원거리 전용 제안은 폐기
- 실제 탄 퍼짐과 화면 Scale의 일치는 별도 작업


## 근거와 관련 문서

- 대화: PlayerHUD 작업 / 장비와 인벤토리 구현 방향 확인
- [대화·문서·코드 확인 범위](../../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
- [앵커와 DPI](%EC%95%B5%EC%BB%A4%EC%99%80%20DPI.md)
- [텍스트와 아이콘 오류](../../Troubleshooting/UMG%20%ED%85%8D%EC%8A%A4%ED%8A%B8%EC%99%80%20%EC%95%84%EC%9D%B4%EC%BD%98%20%ED%91%9C%EC%8B%9C%20%EC%98%A4%EB%A5%98.md)
- [Astra 무기 조준·반동·명중률](../../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/%EB%AC%B4%EA%B8%B0%20%EC%A1%B0%EC%A4%80%C2%B7%EB%B0%98%EB%8F%99%C2%B7%EB%AA%85%EC%A4%91%EB%A5%A0.md)
- [Unreal 지식 목록](../Unreal%20%EC%A7%80%EC%8B%9D%20%EB%AA%A9%EB%A1%9D.md)
- [99_작업 규칙](../../../99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99/99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99.md)
