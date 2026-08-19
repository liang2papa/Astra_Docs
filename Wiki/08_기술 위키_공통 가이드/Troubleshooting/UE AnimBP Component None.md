# UE - AnimBP Component None

## 증상

Anim Blueprint에서 WeaponComponent 참조가 `None`이다.

## 원인

PlayerCharacter의 Component 참조가 Blueprint에 필요한 수준으로 노출되지 않았거나 조회 경로가 연결되지 않았다.

## 해결

`GetComponentByClass`로 Component를 찾고 PlayerCharacter의 Blueprint 노출 설정을 조정한다.

## 관련

- [[무기 시스템]]
- [[애니메이션과 조준]]

