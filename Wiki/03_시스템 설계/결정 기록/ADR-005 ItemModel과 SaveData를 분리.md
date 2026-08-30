# ADR-005 ItemModel과 SaveData를 분리

## 상태

Accepted

## 문제

플레이어 소유 아이템은 레벨을 넘어 상태가 유지되어야 하지만, 바닥 아이템과 현재 레벨의 UObject까지 모두 영속시키면 런타임 수명과 저장 책임이 결합된다. Item ID를 세이브에 그대로 보존하면 새 레벨의 ID 발급 체계와도 충돌할 수 있다.

## 결정

정적 정의는 Item DataAsset, 현재 World의 객체는 ItemInstance, 레벨 간 플레이어 소유 상태는 Item SaveData로 분리한다. ADR 작성 당시의 `ItemModel` 개념은 코드에서 `UItemInstance`라는 이름으로 구현됐다.

ItemInstance의 ID는 현재 World에서만 유효한 Runtime Item ID다. `UItemInstanceFactorySubsystem`이 생성과 ID 발급을 담당한다. 레벨 전환 시 플레이어 Inventory와 Gear 상태를 SaveData로 직렬화하고, 새 World에서 새 ItemInstance와 Runtime Item ID를 생성한다. 바닥 아이템은 레벨 종료 시 폐기하며 글로벌 Item Registry는 두지 않는다.

## 이유

- UObject 수명과 저장 포맷을 분리할 수 있다.
- 레벨별 런타임 ID 충돌을 세이브 영역으로 가져오지 않는다.
- 플레이어 소유 상태만 선택적으로 유지할 수 있다.
- ItemInstance 구조가 바뀌어도 SaveData 변환 규칙을 별도로 관리할 수 있다.

## 보류

- 저장 데이터 버전 관리와 마이그레이션 방식
- Item SaveData Schema와 복원 순서

---

## 상위 페이지

- [ADR 목록](ADR%20%EB%AA%A9%EB%A1%9D.md)
- [03_시스템 설계](../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84.md)

## 관련

- [장비와 인벤토리](../%EC%9E%A5%EB%B9%84%EC%99%80%20%EC%9D%B8%EB%B2%A4%ED%86%A0%EB%A6%AC.md)
- [Item](../%EA%B0%9D%EC%B2%B4%20%EC%84%A4%EA%B3%84/Item.md)
- [시스템 아키텍처](../%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98.md)
