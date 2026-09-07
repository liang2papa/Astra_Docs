[데이터 구조]
Item
 │
 └── Ammunition

■ 탄약 고유 ID, 호환 종류, 발사체 동작을 함께 정의
ItemID: 아이템 고유 ID

AmmoTypeTag: 총기에 호환되는 총알 종류 태그

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

[DataAsset에서 직접 설정]
ProjectileActorClass: ProjectileFactorySubsystem이 생성할 발사체 Actor Class
ProjectileMesh: 발사체 Mesh
TrailVFX: 발사체 Trail 효과
DefaultImpactVFX: 기본 충돌 효과
DefaultImpactSound: 기본 충돌 효과음
DefaultImpactDecal: 기본 충돌 Decal
RicochetImpactVFX: 도탄 충돌 효과
PenetrationEntryVFX: 관통 진입 효과
PenetrationExitVFX: 관통 이탈 효과
ExplosionVFX: 폭발 효과
