[데이터 구조]
Item
 │
 └── Ammunition
 		│
  		└ Projectile	

■ 탄약 고유 ID와 총알 종류를 정의
ItemID: 아이템 고유 ID

AmmoTypeTag: 총기에 호환되는 총알 종류 태그

ProjectileID: 어떠한 Actor가 생성되는지, Actor의 고유 ID

ProjectileID: 액터의 고유 ID

Speed: 총알이 날아가는 속도	

Lifetime: 총알의 생존 시간, 생존시간이 다 되면 풀에 돌아감

GravityScale: 중력의 영향을 받는가? 기본적으로 0이지만 유탄 같은 것은 받을 것임

CollisionRadius: 총알의 충돌 범위

[미구현]
MaxRicochetCount: 도탄 횟수
RicochetVelocityRetention: 도탄 속도	
RicochetDamageRetention: 도탄 데미지 감소율	

[미구현]
bExplosive: 폭발형 여부
ExplosionRadius: 폭발 범위	
bHoming: 유도 여부
HomingAcceleration: 유도 시 가속	

[미구현]
ImpactDecalSize: 충돌 시 이펙트 크기	
ImpactDecalLifetime: 이펙트 지속 시간
