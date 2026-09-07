[데이터 구조]
Item
 │
 ┝ ─ Ammunition
 │
 ┝ ─ Equipment
 │		│
 │		┝ Accessory
 │		┝ Armor
 │		┝ Bag
 │		└ [Weapon]
 │			│
 │			├ WeaponAccuracy: 명중률 설정
 │			├ WeaponAim: 조준 설정
 │			├ WeaponAmmo: 탄약 관련 설정
 │			├ WeaponCharge: 재장전 설정
 │			├ WeaponContinuous: 레이저와 같은 지속형 무기의 타이밍 설정
 │			├ WeaponDamage: 피해량 관련 설정
 │			├ WeaponFire: 공격 입력 시 실제 발사 타이밍 설정
 │			└ WeaponRecoil: 반동 설정
 ┝ ─ Ingredient
 ┝ ─ Miscellaneous
 └ ─ QuestItem

/////////////////////////////
[01_Weapon.csv]
■ 무기 기본 설정
ItemID: 아이템 고유 ID	

WeaponType: 무기 타입 (Pistol, AR, SR, Club, SG, Granade Launcher 등)

AttackKind: 근거리/원거리 구분

DeliveryType: 발사 타입, 원거리일 경우에만 유효 (투사체, 히트스캔 등)



////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
[WeaponAccuracy.csv]
■ 무기 명중률 설정

ItemID: 아이템 고유 ID	
BaseAccuracy: 기본 명중률	
AimingAccuracyBonus: 조준 시 명중률 보정	
MovingHipFireAccuracyPenalty	: 이동 중 명중률 보정
MovingAimingAccuracyPenalty	: 이동 중 조준 시 명중률 보정
AirborneHipFireAccuracyPenalty: 점프, 하강 중 명중률 보정	
AirborneAimingAccuracyPenalty: 점프, 하강 중 조준 시 명중률 보정	
MaxSpreadAngleDegrees: 최대 탄퍼짐 각도
	
[미구현, 기획 필요]
AIAccuracyMultiplier: AI 명중률 판정
AIMovingAccuracyPenalty: AI가 이동 중 명중률 패널티
AITargetMovingAccuracyPenalty: AI가 이동중인 타겟에 대한 명중률 패널티
AIMissSpreadAngle: AI 탄퍼짐 각도

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

[WeaponAim.csv]
■ 무기 조준 설정

ItemID: 아이템 고유 ID	
AimInputMode: 조준 버튼 모드 (누르고 있기, 토글 형식)
AimFOV: 조준 시 확대되는 이동 카메라 시야각
AimFOVInterpSpeed: 조준 시 확대되는 보간속도
VisualAimCorrectionInterpSpeed: 조준 시 총구가 조준점으로 향하는 보간속도 (애니메이션 관련 값)
[이 각도 이상으로는 조준점이 과도하게 보정되지 않음]
MaxVisualAimCorrectionPitch: 	조준 시 총구가 조준점으로 향하는 최대 Pitch 보정 각도
MaxVisualAimCorrectionYaw: 조준 시 총구가 조준점으로 향하는 최대 Yaw각도


////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

[WeaponAmmo.csv]
■ 무기가 사용하는 탄약 설정

ItemID: 아이템 고유 ID	
bUsesAmmo: 탄약을 사용하는 무기인가? (탄약이 없는 근접 무기나 원거리 무기도 있을 수 있음) 	
AmmoTypeTag: 이 무기와 호환되는 탄약 종류	
MaxLoadedAmmo	AmmoPerShot: 한 발 당 몇 발씩 나가는지	
ReloadTime: 재장전 시간
BurstShotCount: 버스트 사격 시 한 번에 발사하는 총알 수

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

[WeaponCharge.csv]
■ 충전식 무기 설정, 충전식 무기인 경우에만 적용
■ 미구현

ItemID: 아이템 고유 ID	
MinChargeHoldTime: 최소 충전 시간 	
MaxChargeTime: 최대 충전 시간	
ChargeAmmoConsumeInterval	: 충전 중 탄약소비 간격
ChargeAmmoPerTick: 틱 당 탄약 소비
MinChargeDamageMultiplier: 최소 충전 시 피해량 배율
MaxChargeDamageMultiplier: 최대 충전 시 피해량 배율

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

[WeaponContinuous.csv]
■ 지속 피해 무기 설정, 지속 무기인 경우 적용
■ 아직 미구현

ItemID: 아이템 고유 ID	
ContinuousTickInterval: 지속 피해 간격, 몇 틱에 피해를 줄 것인지?
InitialContinuousAmmoCost: 발사 시작 시 사용되는 탄약 수, 옵치 자리야가 레이저를 쏠 때의 느낌)
ContinuousAmmoCostIncreasePerTick: 틱 당 탄약 소비량 증가율, 0이면 고정되어 사용
InitialContinuousDamageMultiplier: 초기 발사 시 피해량 배율	
ContinuousDamageMultiplierIncreasePerTick: 틱 당 피해량 증가 배율, 0이면 피해량 증가 없음
MaxDamageMultiplierIncreasePerTick: 틱 당 피해량 증가 배율 최대치, 자리야 빨간 레이저 느낌으로 생각하면 됨

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

[WeaponDamage.csv]
■ 무기의 피해량과 유효 사거리에 따른 피해량 감소 데이터

ItemID: 아이템 고유 ID	
BaseDamage: 기본 피해량	
CriticalChancePercent: 기본 치명 확률	
CriticalDamageMultiplier	: 기본 치명 배율, 1이면 치명 피해량 없음, 1.5면 150%
EffectiveRange: 유효 사거리
OutOfRangeDamageMultiplier: 유효 사거리 밖에서의 피해 감소율, 0.8이면 실제 20%의 피해만 줌

■ 아직 미구현
PierceProbabilityPercent: 관통 확률, 탄이 적과 닿았을 때 사라지지 않고 이어서 갈 확률, 몬헌의 관통탄

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

[WeaponFire.csv]
■ 발사 입력 후 실제 발사 타이밍 설정값 

ItemID: 아이템 고유 ID	
ShootType: 발사 타입 (단발, 연발, 지속, 충전식)	
AttackCooldown: 총기 발사 후 다음 발사까지의 대기시간
FirstFireDelayTime: 처음 무기를 들고 조준 자세를 취할 때 까지의 시간
AutoHolsterDelay: 공격 조작이 없을 경우 무기를 집어넣는 애니메이션 딜레이	
WeaponDrawAlignmentToleranceYaw: 무기를 꺼내는 애니메이션 재생에 필요한 회전 위치
DrawReadyDelay: 무기를 꺼낼 때까지 대기하는 시간	

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

[WeaponRecoil.csv]
■ 무기 반동 설정값 
ItemID: 아이템 고유 ID	
bUsesRecoil	: 반동이 있는 무기인가?

// 무기 발사 당 수치
Min, Max 사이에서 무작위로 반동값 생성
RecoilPitchMin: 최소 반동 pitch
RecoilPitchMax: 최대 반동 pitch	
RecoilYawMin: 최소 반동 yaw	
RecoilYawMax: 최대 반동 yaw

// 무기 
MaxRecoilPitch: 최대 반동 pitch
MaxRecoilYaw: 최대 반동 yaw	
RecoilRecoveryDurtion: 반동 회복 속도, 무기가 반동을 마친 후 원래 자리로 돌아오는 시간