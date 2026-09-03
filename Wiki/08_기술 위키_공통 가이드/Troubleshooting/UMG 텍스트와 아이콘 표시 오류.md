# UMG 텍스트와 아이콘 표시 오류

## 설계 의도

## 아이콘이 안 보이는 경우

- 확인된 사례: 작은 PIE 화면에서 Stretch Anchor로 아이콘 영역 축소
- 데이터·이벤트는 정상. 단일 점 Anchor와 고정 요청 크기로 배치 수정
- 화면 Print String은 최신 메시지가 위에 추가될 수 있음
- 화면의 줄 위치만으로 마지막 실행 상태를 단정하지 않음
- 시간순 Output Log → 표시 이벤트 인자 → Texture → Visibility → 부모 크기·Anchor → DPI 순서 확인

## 텍스트 사각형과 LastResort

- 확인된 사례: HUD Rich Text Block의 유효하지 않은 폰트 설정
- Designer → 텍스트 선택 → Appearance → Default Text Style Override → Font 확인
- 유효한 Font Family / Typeface 지정 후 Compile·Save·PIE
- 숫자만 표시할 때 일반 Text Block도 사용 가능
- 다른 사례: On-screen Debug Message의 이모지 글리프 부재
- 로그의 대상 텍스트·코드포인트 확인 후 디버그 문자열 또는 해당 폰트 수정
- HUD 폰트 문제와 C++ 디버그 메시지 문제를 구분

## 전체 확인 순서

```text
표시 이상 발견
→ Output Log에서 발생 위젯·문자·시각 확인
  ├─ Image → 이벤트 인자 → Texture → Visibility → Anchor / Size / DPI
  └─ Text → 실제 표시 문자열 → 위젯 Font / Rich Text Style 확인
           └─ Debug Message라면 해당 C++ 문자열·디버그 폰트 확인
→ 한 원인씩 수정 → Compile / Save → PIE 비교
→ 정상 표시 사용자 확인 후 해결 기록
```

## 근거와 관련 문서

- 대화: PlayerHUD 작업 — 아이콘 축소·폰트 정상 표시 확인
- [대화·문서·코드 확인 범위](../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
- [앵커와 DPI](../Unreal/UMG/%EC%95%B5%EC%BB%A4%EC%99%80%20DPI.md)
- [HUD 값과 아이콘 표시](../Unreal/UMG/HUD%20%EA%B0%92%EA%B3%BC%20%EC%95%84%EC%9D%B4%EC%BD%98%20%ED%91%9C%EC%8B%9C.md)
- [Unreal 지식 목록](../Unreal/Unreal%20%EC%A7%80%EC%8B%9D%20%EB%AA%A9%EB%A1%9D.md)
- [99_작업 규칙](../../99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99/99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99.md)
