[아이템 데이터 구조]
[Item]
 │
 ├── Ammunition
 │
 ├── Equipment
 │     │
 │     ┠ Weapon
 │     │   │
 │     │   ├── WeaponDamage
 │     │   ├── WeaponFire
 │     │   ├── WeaponAmmo
 │     │   ├── WeaponAccuracy
 │     │   ├── WeaponAim
 │     │   ├── WeaponRecoil
 │     │   ├── WeaponCharge
 │     │   └── WeaponContinuous
 │     │
 │     ┠ Armor
 │     │
 │     ┠ Accessory
 │     │
 │     └ Bag
 │ 	 
 ├── Ingredient       
 │
 ├── Miscellaneous
 │
 └── QuestItem



[공통]
■ ItemID
□ 고유 ID, 언리얼에서는 이 값을 숫자로 쓸 필요가 없었음

[ID 작성 예시]
방식: 장비 구분, 아이템 고유 이름
무기의 경우
예시) Equipment.Weapon.BasePistol

총알의 경우
예시) Ammunition.Base9mm

[01_Item.csv]

ItemID: 아이템 고유 ID	

ItemCategory: 아이템 분류 (탄약, 장비, 재료, 기타, 퀘스트 아이템)	

ItemRarity: 희귀도: (Common, Rare, Unique, Relic)	

MaxStackSize: 최대 소지 수량 수, 1이면 스택 불가, 1보다 크면 스택 가능

Weight: 아이템 무게	

bConsumable: 플레이어가 인벤토리창에서 사용 가능한 아이템인가?

bCanBeCraftingMaterial: 재료로 사용 가능한가?

bDismantleable: 분해 가능한가?
