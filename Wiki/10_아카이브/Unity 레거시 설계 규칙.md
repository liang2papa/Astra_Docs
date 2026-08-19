# Unity 레거시 설계 규칙

> 이 문서는 Unity 구현의 설계 의도를 보존하기 위한 참고 자료입니다. 현재 Unreal Engine 5.8 구현과 충돌하면 Unreal Wiki 본문을 우선합니다.

## 1. 이전 계층

| Unity 계층 | 이전 역할 | Unreal 대응 방향 |
|---|---|---|
| Lv0 Installer | DI Container 생성, 하위 객체 조립 | 현재 별도 계층 없음. Actor/Component 생명주기와 GameMode/GameInstance 사용 |
| Lv1 Service | Input, Audio, SaveLoad, Options | 필요 시 GameInstance Subsystem 또는 Engine/LocalPlayer Subsystem |
| Lv2 Director | 씬 규칙과 흐름 | GameMode, GameState, Level Script 또는 전용 World Subsystem |
| Lv3 Mediator | 엔티티 하위 객체 중재 | EntityStateMediatorComponent |
| Lv4 Handler/Viewer | 물리·애니메이션·UI 실행 | ActorComponent, AnimInstance, Widget |

전역 Service Container는 현재 만들지 않습니다. Director와 Mediator는 자신에게 전용인 하위 객체를 직접 조립할 수 있다는 기존 원칙을 Unreal 생명주기에 맞춰 적용합니다.

## 2. 유지할 설계 철학

- 정적 데이터와 런타임 모델 분리
- 하위 객체 간 직접 정책 통신 최소화
- Mediator를 통한 이벤트 라우팅
- UI는 Model/Handler 이벤트를 읽고 직접 상태를 결정하지 않음
- 입력 주체와 실행 주체의 분리

## 3. 애니메이션 레거시 규칙

### 어깨 직접 회전 금지

Unity/Mixamo 구현에서는 양쪽 Shoulder Bone을 코드로 독립 회전하면 파지 자세가 무너졌습니다. Unreal에서도 상체 조준을 구현할 때 어깨를 무제한 직접 회전하기보다 Aim Offset, Spine 분배, Control Rig 제한값을 사용합니다.

### 가슴·머리 처리

- Unity에서는 하체 애니메이션에 상체가 끌리지 않도록 Chest 절대 회전을 사용했습니다.
- 무기별 `stanceYawOffset`, `stancePitchOffset`으로 영점을 맞췄습니다.
- Head는 Chest의 자식이므로 동일 오프셋을 중복 적용하지 않았습니다.

Unreal에서는 이 목적을 Aim Offset, Layered Blend Per Bone, Control Rig로 재설계합니다.

### Muzzle Alignment

카메라 시차로 탄도가 꺾여 보이는 문제를 막기 위해 Unity에서는 발사 프레임에 무기 루트를 탄도 방향으로 보정했습니다. Unreal 현재 구현은 카메라 Ray로 AimTarget을 구한 뒤 Muzzle에서 AimTarget으로 Projectile 회전을 계산합니다.

## 4. 반동 원칙

- 연사 반동에 카메라 Impulse를 사용하지 않습니다.
- Pitch/Yaw 오프셋을 보간하고 회복시켜 플레이어가 스프레이를 제어할 수 있게 합니다.
- Camera Shake/Impulse는 피격·폭발 같은 외부 충격에 사용합니다.
- 무기별 Snappiness, ReturnSpeed, 최대 반동값은 DataAsset으로 조정합니다.

## 5. 물리·접지 레거시

Unity에서는 Idle과 이동 상태에 따라 PhysicMaterial 마찰력을 교체하고 SphereCast로 접지를 검사했습니다. Unreal에서는 CharacterMovement의 Ground Friction, Braking, Floor Check, MovementMode를 사용하며 Unity 수치를 그대로 복사하지 않습니다.

## 6. Soul-Link 레거시

이전 Soul-Link 유형:

- Possession
- DirectSkill
- Transform 또는 Enhancement
- Totem

유지할 의도:

- 빙의 시 NPC의 고유 상태 머신과 능력을 보존합니다.
- 플레이어 입력 주체만 교체합니다.
- 통제권을 잃은 원래 Pawn도 복귀·취소 조건을 처리할 수 있어야 합니다.
- Jump/Evade 같은 부적절한 자세에서는 빙의 진입을 막습니다.
- 변신 중 무기 사용을 차단합니다.
- 변신 해제 후 Soul-Link 쿨다운은 유지합니다.

Unreal에서는 PlayerController Possess, PlayerState ASC, NPC StateTree/ASC를 조합해 구현합니다.

## 7. 렌더링 레거시

Unity URP의 Shadow Atlas 규칙은 그대로 적용하지 않습니다. 다만 다음 성능 원칙은 유지합니다.

- 동적 그림자 광원 수 최소화
- 정적 환경은 가능한 경우 베이크 또는 Unreal의 적절한 정적 조명 전략 사용
- Shadow Map/Virtual Shadow Map 경고를 설정 증가만으로 덮지 말고 광원 수와 영향 범위를 먼저 줄임
- 보조 이펙트 조명의 그림자 필요성 검토
