# Troubleshooting 목록

## 설계 의도

증상·원인·해결·검증 근거 축적. 미해결과 해결 완료 구분

## 일반 오류·UI 해결 사례

- [UE C2027 Incomplete Type](UE%20C2027%20Incomplete%20Type.md)
- [UE Interface IntelliSense False Positive](UE%20Interface%20IntelliSense%20False%20Positive.md)
- [UE AnimBP Component None](UE%20AnimBP%20Component%20None.md)
- [UMG 텍스트와 아이콘 표시 오류](UMG%20%ED%85%8D%EC%8A%A4%ED%8A%B8%EC%99%80%20%EC%95%84%EC%9D%B4%EC%BD%98%20%ED%91%9C%EC%8B%9C%20%EC%98%A4%EB%A5%98.md)
- [Stack 분할이 1개로 고정 — 멤버 저장 누락](../Unreal/UMG/%EC%88%98%EB%9F%89%20%EC%9E%85%EB%A0%A5%EA%B3%BC%20Stack%20%EB%B6%84%ED%95%A0.md)
- [장착 옵션이 Head로 고정 — enum 초기화 누락](../Unreal/UMG/%EB%8F%99%EC%A0%81%20%EC%9C%84%EC%A0%AF%20%EC%83%9D%EC%84%B1%EA%B3%BC%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EA%B0%B1%EC%8B%A0.md)
- [팝업 화면 이탈·반대 방향 이동](../Unreal/UMG/%ED%8C%9D%EC%97%85%20%EC%9C%84%EC%B9%98%EC%99%80%20%ED%99%94%EB%A9%B4%20%EA%B2%BD%EA%B3%84.md)
- [발사 시 조준 Pose 덮임·중복 FABRIK](../Unreal/Animation/Slot%C2%B7AimOffset%C2%B7IK%20%EC%A0%81%EC%9A%A9%20%EC%88%9C%EC%84%9C.md)

## Astra 기술 이슈

- [애니메이션 — 현재 기록·해결·검증](Astra%20%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98%20%EC%9D%B4%EC%8A%88%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)
- [로코모션 — 현재 기록·해결·검증](Astra%20%EB%A1%9C%EC%BD%94%EB%AA%A8%EC%85%98%20%EC%9D%B4%EC%8A%88%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)
- [무기·전투 — 현재 기록·해결·검증](Astra%20%EB%AC%B4%EA%B8%B0%C2%B7%EC%A0%84%ED%88%AC%20%EC%9D%B4%EC%8A%88%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)
- [아이템·인벤토리 — 현재 기록·해결·검증](Astra%20%EC%95%84%EC%9D%B4%ED%85%9C%C2%B7%EC%9D%B8%EB%B2%A4%ED%86%A0%EB%A6%AC%20%EC%9D%B4%EC%8A%88%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)
- [구조·GAS — 현재 기록·해결·검증](Astra%20%EA%B5%AC%EC%A1%B0%C2%B7GAS%20%EC%9D%B4%EC%8A%88%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)

기존 기록 전체 이관. 원인이 확정되지 않은 항목은 검증 대기 유지

## 후속 검증 순서

기존 남은 이슈의 권장 순서 이관. 완료된 일반 Stack 분할은 실패·취소 회귀로 범위 축소

1. 장착 슬롯 조합과 Stack 분할 실패·취소 경로 확인
2. Inventory/Gear Drop·WorldItem Pool 재사용·실패 경로 확인
3. 최신 소스 전체 빌드와 Pawn 교체 회귀 검증
4. Bag Capacity 감소·초과 WorldItem Transaction 구현
5. SaveData Schema·레벨 전환 복원 구현
6. Preferred/Reserved 탄약 UI 연결 후 RapidFire·명중률 연동

## 일정 상태·상위 페이지

- [남은 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%EB%82%A8%EC%9D%80%20%EC%9D%B4%EC%8A%88.md)
- [해결된 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%ED%95%B4%EA%B2%B0%EB%90%9C%20%EC%9D%B4%EC%8A%88.md)
- [08_기술 위키_공통 가이드](../08_%EA%B8%B0%EC%88%A0%20%EC%9C%84%ED%82%A4_%EA%B3%B5%ED%86%B5%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)
- [정리 근거](../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
