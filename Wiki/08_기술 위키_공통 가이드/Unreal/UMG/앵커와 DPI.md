# 앵커와 DPI

## 설계 의도

작은 화면에서도 HUD 아이콘의 위치와 식별 가능한 크기 유지

## 서로 다른 세 값

- Anchor: 부모 Canvas에서 위치의 기준점 또는 범위
- Alignment: 자신의 사각형에서 기준점으로 사용할 비율
- DPI Scale: 해상도에 따른 UI 전체 배율

Min/Max Anchor가 같으면 단일 점, 다르면 해당 축 Stretch

Stretch는 배경·확장 영역에 사용 가능. 고정 크기 아이콘에 일괄 적용하는 규칙은 아님

## 배치 예시

아래 수치는 대화의 기능 검증용 예시. 최종 아트 수치 아님

| 대상 | Anchor | Alignment | 크기·위치 |
|---|---|---|---|
| 체력 묶음 | 좌하단 (0,1) | (0,1) | Size 360×32 / Position (40,-40) |
| 무기 아이콘 | 우하단 (1,1) | (1,1) | Size 예: 96×96 / 여백은 음수 Offset |
| 크로스헤어 | 중앙 (0.5,0.5) | (0.5,0.5) | Position (0,0) |

중앙 확대 기준: Render Transform Pivot (0.5,0.5)

Canvas Slot 크기와 SizeBox의 값은 UI 단위. DPI 적용 후 물리 픽셀과 다를 수 있음

## 설정 위치

- Widget Blueprint → Designer → 위젯 선택 → Details → Slot (Canvas Panel Slot)
- 프로젝트 DPI: Edit → Project Settings → Engine → User Interface → DPI Scaling
- Designer의 Screen Size: 해상도·화면비 미리보기

앵커 기준은 [Epic Anchors](https://dev.epicgames.com/documentation/en-us/unreal-engine/umg-anchors-in-unreal-engine-ui), 배율 설정은 [Epic DPI Scaling](https://dev.epicgames.com/documentation/en-us/unreal-engine/dpi-scaling-in-unreal-engine) 참고

## 확인 순서

```text
화면 위치 기준 Anchor 지정
→ Alignment 지정
→ Position / Size 지정
→ Image의 Set Brush from Texture: Match Size=false
→ 1280×720 / 1920×1080 / 작은 PIE 창 / 넓은 화면비 확인

이미지가 지나치게 작음
→ Stretch 범위와 Offset 확인
→ 부모 영역 크기 확인
→ DPI Scale 확인
→ Brush / Visibility / Texture 확인
```

## 근거와 관련 문서

- 대화: PlayerHUD 작업 — 작은 화면의 아이콘 축소 원인 사용자 확인
- [대화·문서·코드 확인 범위](../../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
- [팝업 위치와 화면 경계](%ED%8C%9D%EC%97%85%20%EC%9C%84%EC%B9%98%EC%99%80%20%ED%99%94%EB%A9%B4%20%EA%B2%BD%EA%B3%84.md)
- [텍스트와 아이콘 표시 오류](../../Troubleshooting/UMG%20%ED%85%8D%EC%8A%A4%ED%8A%B8%EC%99%80%20%EC%95%84%EC%9D%B4%EC%BD%98%20%ED%91%9C%EC%8B%9C%20%EC%98%A4%EB%A5%98.md)
- [Unreal 지식 목록](../Unreal%20%EC%A7%80%EC%8B%9D%20%EB%AA%A9%EB%A1%9D.md)
- [99_작업 규칙](../../../99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99/99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99.md)
