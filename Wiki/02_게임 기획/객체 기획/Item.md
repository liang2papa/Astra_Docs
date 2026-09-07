# Item

## 기획 의도

## 정의

플레이어 또는 다른 Entity가 획득·소유·보관·이전할 수 있는 객체

## 종류

| 종류 | 정의 |
|---|---|
| Equipment | 슬롯에 장착하여 사용하는 장비 |
| Ammunition | 무기에 사용 권한을 등록하는 탄약 |
| Ingredient | 제작·수리·강화 재료가 주목적인 아이템 |
| QuestItem | 임무 진행·조건 판정이 주목적인 아이템 |
| Miscellaneous | 다른 종류에 속하지 않는 일반 아이템 |

장비 종류: Weapon · Armor · Accessory · Bag

## 종류와 속성

- 대표 종류: 하나
- 추가 속성: 여러 개 조합 가능
- Consumable: 사용 시 수량·내구도 소비
- Craftable: 제작을 통한 생성 가능
- Crafting Ingredient: 다른 아이템의 재료로 사용 가능
- Stackable · Tradeable · Droppable · Unique · Quest: 개별 속성

| 사례 | 대표 종류 | 추가 속성 |
| --- | --- | --- |
| 제작 가능한 총기 | Equipment / Weapon | Craftable · Tradeable · Droppable |
| 일반 탄환 | Ammunition | Consumable · Craftable · Stackable |
| 재료로도 사용하는 특수 탄환 | Ammunition | Consumable · Craftable · Crafting Ingredient |
| 철광석 | Ingredient | Crafting Ingredient · Stackable · Tradeable |
| 퀘스트 전용 열쇠 | QuestItem | Quest · Unique |

## 품질

Common → Rare → Unique → Relic

## 탄약과 투사체

- Ammunition: 획득·소유·보관되는 탄약
- Projectile: 공격 후 이동·충돌하는 투사체
- 맨손 공격: Item이 아닌 Entity의 전투 행동

## 문서 연결

- 기획: [Item](Item.md)
- 시스템 설계: [Item](../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/%EA%B0%9D%EC%B2%B4%20%EC%84%A4%EA%B3%84/Item.md)
- 애니메이션 설계: [Item](../../04_%EC%95%84%ED%8A%B8%20%EB%B0%8F%20%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98/%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98%20%EC%84%A4%EA%B3%84/Item.md)
