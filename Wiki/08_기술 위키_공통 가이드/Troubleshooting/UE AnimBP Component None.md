# UE - AnimBP Component None

## 증상

Anim Blueprint에서 WeaponComponent 참조가 `None`이다.

## 원인

PlayerCharacter의 Component 참조가 Blueprint에 필요한 수준으로 노출되지 않았거나 조회 경로가 연결되지 않았다.

## 해결

`GetComponentByClass`로 Component를 찾고 PlayerCharacter의 Blueprint 노출 설정을 조정한다.

---

## 상위 페이지

- [Troubleshooting 목록](Troubleshooting%20%EB%AA%A9%EB%A1%9D.md)
- [08_기술 위키_공통 가이드](../08_%EA%B8%B0%EC%88%A0%20%EC%9C%84%ED%82%A4_%EA%B3%B5%ED%86%B5%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)

## 관련

- [무기 시스템](../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/%EB%AC%B4%EA%B8%B0%20%EC%8B%9C%EC%8A%A4%ED%85%9C.md)
- [애니메이션과 조준](../../04_%EC%95%84%ED%8A%B8%20%EB%B0%8F%20%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98/%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98%EA%B3%BC%20%EC%A1%B0%EC%A4%80.md)

