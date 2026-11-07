# Astra SoulLink — 프로젝트 맵 (PROJECT MAP)

> 파일·클래스 탐색을 줄이기 위한 경로 매핑 파일.
> 경로 갱신: 2026-10-20 Entity/NPC/Animation 보완. 그 외 목록은 2026-09-16 스캔 기준이므로 사용 전 파일 존재 확인
> 문서 진입점 정리: 2026-10-21. 현재 변경·연결·검증 상태는 [현재 작업 상태](Wiki/01_일정%20관리/현재%20작업%20상태.md)에서 먼저 확인

---

## 1. 프로젝트 루트 경로

| 구분 | 경로 |
|---|---|
| Unreal 프로젝트 | `D:\FruitsfamilyStudio\Projects\Astra_Unreal` |
| 문서 (Docs) | `D:\FruitsfamilyStudio\Projects\Astra_Docs` |
| 프로젝트 파일 | `D:\FruitsfamilyStudio\Projects\Astra_Unreal\Astra_SoulLink.uproject` |
| 솔루션 | `D:\FruitsfamilyStudio\Projects\Astra_Unreal\Astra_SoulLink.sln` |
| C++ 소스 루트 | `D:\FruitsfamilyStudio\Projects\Astra_Unreal\Source` |

## 2. 작업 재개와 규칙

- [작업 규칙](Wiki/99_작업%20규칙/99_작업%20규칙.md)과 [현재 작업 상태](Wiki/01_일정%20관리/현재%20작업%20상태.md)를 먼저 읽고 관련 설계·QA 문서와 실제 소스를 대조.
- C++ 수정은 변경안·영향 설명과 사용자 승인 범위에 맞춰 진행. Blueprint·StateTree·AnimBP·DataAsset 편집과 빌드·프로젝트 파일 재생성은 사용자 수행.
- 기존 파일은 변경 부분을 확인해 보존하며 작업 전후 diff 검토. 이 경로 맵 자체를 수정·빌드 승인으로 사용하지 않음.

## 3. Unreal 프로젝트 주요 폴터

| 폴터 | 내용 |
|---|---|
| `Source\Astra_SoulLink` | 게임 런타임 C++ 코드 |
| `Source\Astra_SoulLinkEditor` | 에디터 전용 코드 (데이터 임포트 등) |
| `Config` | 엔진/프로젝트 설정 |
| `Content` | 에셋 |
| `Data_CSV` | 데이터 테이블 CSV |
| `Plugins` | 플러그인 |

## 4. 클래스 → 파일 매핑

경로는 `Source\Astra_SoulLink\` 기준. `.cpp`는 동일 폴터의 같은 이름 파일.

### 코어
| 클래스 | 헤더 |
|---|---|
| Astra_SoulLinkCharacter | `Astra_SoulLinkCharacter.h` |
| Astra_SoulLinkGameMode | `Astra_SoulLinkGameMode.h` |
| Astra_SoulLinkPlayerController | `Astra_SoulLinkPlayerController.h` |
| ReadMeDataAsset | `ReadMeDataAsset.h` |

### Ability (Gameplay Ability)
| 클래스 | 헤더 |
|---|---|
| AstraEntityGameplayAbilityBase | `Ability\AstraEntityGameplayAbilityBase.h` |
| AstraWeaponGameplayAbilityBase | `Ability\AstraWeaponGameplayAbilityBase.h` |
| AstraGameplayTags | `Ability\AstraGameplayTags.h` |

### Effect
| 클래스 | 헤더 |
|---|---|
| CombatEffectSubsystem | `Effect\CombatEffectSubsystem.h` |

### Entity — Ability
| 클래스 | 헤더 |
|---|---|
| EntityAbilitySystemInitializer | `Entity\Ability\EntityAbilitySystemInitializer.h` |
| EntityAttributeSet | `Entity\Ability\EntityAttributeSet.h` |
| EntityStaminaAttributeSet | `Entity\Ability\EntityStaminaAttributeSet.h` |
| EntityStaminaComponent | `Entity\Ability\EntityStaminaComponent.h` |
| EntitySoulEnergyComponent | `Entity\Ability\EntitySoulEnergyComponent.h` |

### Entity — Combat / Weapon
| 클래스 | 헤더 |
|---|---|
| WeaponComponent | `Entity\Combat\Weapon\WeaponComponent.h` |
| WeaponAttackTypes | `Entity\Combat\Weapon\WeaponAttackTypes.h` |
| WeaponHandlerContext | `Entity\Combat\Weapon\WeaponHandlerContext.h` |
| WeaponHandlerBase | `Entity\Combat\Weapon\Handlers\WeaponHandlerBase.h` |
| AimHandler | `Entity\Combat\Weapon\Handlers\AimHandler.h` |
| AttackCommitHandler | `Entity\Combat\Weapon\Handlers\AttackCommitHandler.h` |
| AttackSequenceHandler | `Entity\Combat\Weapon\Handlers\AttackSequenceHandler.h` |
| CarryHandler | `Entity\Combat\Weapon\Handlers\CarryHandler.h` |
| RecoilHandler | `Entity\Combat\Weapon\Handlers\RecoilHandler.h` |
| ReloadHandler | `Entity\Combat\Weapon\Handlers\ReloadHandler.h` |

### Entity — Control
| 클래스 | 헤더 |
|---|---|
| ControllableEntity | `Entity\Control\ControllableEntity.h` |
| EntityInputBufferComponent | `Entity\Control\EntityInputBufferComponent.h` |
| EntityInputStateTreeNodes | `Entity\Control\EntityInputStateTreeNodes.h` |
| NPCBrainStateTreeNodes | `Entity\Control\NPCBrainStateTreeNodes.h` |

### Entity — Character / NPC / Animation
| 클래스 | 헤더 |
|---|---|
| EntityCharacter | `Entity\EntityCharacter.h` |
| AstraNPCCharacter | `Entity\NPCCharacter.h` |
| AstraNPCAIController | `Entity\NPCAIController.h` |
| AstraHumanoidAnimInstance / AstraHumanoidAnimationData | `Entity\Animation\AstraHumanoidAnimInstance.h` |
| PlayerContext | `Entity\PlayerContext.h` |
| EntityLocomotionComponent | `Entity\EntityLocomotionComponent.h` |

### Entity — Data / Health / State
| 클래스 | 헤더 |
|---|---|
| AstraAIDefines | `Entity\Data\AstraAIDefines.h` |
| AstraEntityDefines | `Entity\Data\AstraEntityDefines.h` |
| AstraFactionDefines | `Entity\Data\AstraFactionDefines.h` |
| EntityHealthComponent | `Entity\Health\EntityHealthComponent.h` |
| EntityStateEventBus | `Entity\State\EntityStateEventBus.h` |
| EntityStateEventTypes | `Entity\State\EntityStateEventTypes.h` |
| EntityStateMediatorComponent | `Entity\State\EntityStateMediatorComponent.h` |
| EntityContext / Provider / DataAsset / Locomotion | `Entity\EntityContext.h` 등 (Entity 루트) |

### InventorySystem
| 클래스 | 헤더 |
|---|---|
| InventoryComponent | `InventorySystem\InventoryComponent.h` |
| GearComponent | `InventorySystem\GearComponent.h` |
| ItemTransferComponent | `InventorySystem\ItemTransferComponent.h` |
| AmmunitionReservationTypes | `InventorySystem\AmmunitionReservationTypes.h` |
| InventoryMenuComponent / Widget | `InventorySystem\UI\` |

### Item
| 클래스 | 헤더 |
|---|---|
| ItemDataAsset / ItemInstance | `Item\ItemDataAsset.h`, `Item\ItemInstance.h` |
| ItemInstanceFactorySubsystem | `Item\ItemInstanceFactorySubsystem.h` |
| WorldItemActor / Factory | `Item\WorldItemActor.h`, `Item\WorldItemFactorySubsystem.h`, `Item\WorldItemFactorySettings.h` |
| Ammunition (DataAsset/Instance) | `Item\Ammunition\` |
| Equipment (DataAsset/Instance/Types) | `Item\Equipment\` |
| Weapon (DataAsset/Instance) | `Item\Equipment\Weapon\` |

### Player
| 클래스 | 헤더 |
|---|---|
| PlayerCharacter | `Player\PlayerCharacter.h` |
| AstraPlayerController | `Player\AAstraPlayerController.h` |
| AstraPlayerState | `Player\AstraPlayerState.h` |
| AAstraPlayerCameraManager | `Player\AAstraPlayerCameraManager.h` |
| PlayerDataAsset | `Player\PlayerDataAsset.h` |
| InteractionDefines | `Player\Data\InteractionDefines.h` |
| PlayerAttributeSet | `Player\Ability\PlayerAttributeSet.h` |
| PlayerSoulResourceComponent | `Player\Ability\PlayerSoulResourceComponent.h` |
| PlayerHUD (Component/Widget/DataAsset) | `Player\UI\` |
| WeaponCrosshairDataAsset | `Player\UI\WeaponCrosshairDataAsset.h` |

### Projectile
| 클래스 | 헤더 |
|---|---|
| ProjectileActor | `Projectile\ProjectileActor.h` |
| ProjectileFactorySubsystem | `Projectile\ProjectileFactorySubsystem.h` |

### Utils/Defines
`Utils\Defines\` — AstraCombatDefines, EParkourMoveType, ItemTooltipTypes, ItemType, LocomotionActionState, PlayerCrosshairState

### Variant_Combat (전투 모드)
| 클래스 | 헤더 |
|---|---|
| CombatCharacter / CombatGameMode / CombatPlayerController | `Variant_Combat\` 루트 |
| CombatAIController, CombatEnemy, CombatEnemySpawner | `Variant_Combat\AI\` |
| CombatStateTreeUtility, EnvQueryContext_Danger/Player | `Variant_Combat\AI\` |
| AnimNotify_CheckChargedAttack / CheckCombo / DoAttackTrace | `Variant_Combat\Animation\` |
| CombatActivationVolume, CombatCheckpointVolume, CombatDamageableBox, CombatDummy, CombatLavaFloor | `Variant_Combat\Gameplay\` |
| CombatActivatable / CombatAttacker / CombatDamageable | `Variant_Combat\Interfaces\` |
| CombatLifeBar | `Variant_Combat\UI\` |

### Variant_Platforming (플랫폼 모드)
PlatformingCharacter / PlatformingGameMode / PlatformingPlayerController (`Variant_Platforming\` 루트), AnimNotify_EndDash (`Animation\`)

### Variant_SideScrolling (사이드 스크롤 모드)
| 클래스 | 헤더 |
|---|---|
| SideScrollingCharacter / GameMode / PlayerController | `Variant_SideScrolling\` 루트 |
| SideScrollingAIController, SideScrollingNPC, SideScrollingStateTreeUtility | `Variant_SideScrolling\AI\` |
| JumpPad, MovingPlatform, Pickup, SoftPlatform | `Variant_SideScrolling\Gameplay\` |
| SideScrollingInteractable | `Variant_SideScrolling\Interfaces\` |
| SideScrollingCameraManager, SideScrollingUI | 각각 루트, `UI\` |

### Editor (Astra_SoulLinkEditor)
| 클래스 | 헤더 |
|---|---|
| ItemDataAssetImporter | `ItemDataImport\ItemDataAssetImporter.h` |
| ItemDataImportLibrary / Settings | `ItemDataImport\` |

## 5. 문서 (Astra_Docs) 구조

| 폴터 | 내용 |
|---|---|
| `Wiki\00_프로젝트 개요` | 프로젝트 개요 |
| `Wiki\01_일정 관리` | 일정 |
| `Wiki\02_게임 기획` | 기획 |
| `Wiki\03_시스템 설계` | 시스템 설계 |
| `Wiki\04_아트 및 애니메이션` | 아트 |
| `Wiki\05_오디오` | 오디오 |
| `Wiki\06_레벨 디자인` | 레벨 |
| `Wiki\07_QA_테스트` | QA |
| `Wiki\08_기술 위키_공통 가이드` | 기술 가이드 |
| `Wiki\09_릴리즈_패치노트` | 패치노트 |
| `Wiki\10_아카이브` | 아카이브 |
| `Wiki\99_작업 규칙` | 작업 규칙 |
| `기획·설계 대응표.md` | 기획-설계 매핑 |

---

### 사용 예시

- "EntityStaminaComponent에 스태미나 재생 로직 추가해줘" → 경로·현재 구현 확인 → 변경안·영향 설명 → 승인 범위의 소스 수정·diff 검토
- "Combat 모드 AI 구조 설명해줘" → `Variant_Combat\AI\` 파일 읽고 설명
