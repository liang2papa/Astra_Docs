# Unreal Editor 작업 절차

## 1. C++ 변경 후 빌드

### 프로젝트 파일 재생성이 필요하지 않은 경우

- 기존 `.h/.cpp` 내용만 수정
- 새 UPROPERTY/UFUNCTION 추가
- 기존 모듈 안에 파일 추가 후 IDE가 이미 인식

이 경우 `Astra_SoulLink.sln`을 열고 `Development Editor | Win64`로 빌드합니다.

### 프로젝트 파일 재생성이 필요한 경우

- `.sln`이 없거나 손상됨
- 새 모듈/플러그인 추가
- `Build.cs`, `Target.cs`, 모듈 의존성 변경
- 새 C++ 파일이 IDE에 나타나지 않음

`Astra_SoulLink.uproject` 우클릭 → **Generate Visual Studio project files** 후 솔루션을 빌드합니다.

구조 변경 후에는 Live Coding만으로 확인하지 말고 Editor를 닫은 상태에서 전체 Editor Target을 빌드합니다.

## 2. Blueprint Context 바인딩

STC/STT가 `PlayerContext` 또는 `EntityContext`를 읽을 때 Blueprint 변수만 만들면 값이 자동으로 들어오지 않습니다.

```text
StateTree Asset
→ 해당 Condition/Task 선택
→ Details > Binding
→ Actor.EntityContext 또는 Actor.PlayerContext 연결
→ Compile
→ 런타임 Debug Object에서 값 확인
```

`None` 오류가 나면 Cast를 추가하기 전에 다음을 확인합니다.

1. Character에서 Context가 실제 생성·초기화되었는가
2. StateTree 시작이 Context 초기화보다 빠르지 않은가
3. Condition/Task의 인스턴스 바인딩이 현재 StateTree에 저장되었는가
4. 부모 Task/Condition 변수와 자식의 동일 이름 변수가 중복되지 않았는가

## 3. STC_CanFire

권장 전체 순서:

```text
ReceiveTestCondition
→ IsValid(EntityContext)?
→ IsValid(EntityContext.WeaponComp)?
→ IsValid(EntityContext.StatusModel 또는 ASC)?
→ Dead/Stunned/Traversal 차단 확인
→ UpperState == Ready 확인
→ BaseState == Idle OR Walk 확인
→ WeaponComp.CanFireWeapon
→ 모든 결과 AND
→ Return true

유효성 실패 또는 조건 실패
→ Return false
```

Sprint 중 Fire 입력은 STC_CanFire에서 바로 허용하지 않고, 먼저 CancelSprint/Walk 전이와 Draw/Ready 흐름을 거친 뒤 Fire로 진입합니다.

## 4. STT_Fire

```text
Event EnterState
→ IsValid(EntityContext.StateEventBus)
→ Publish Simple
   EventTag = Event.State.Upper.FireEntered
   Source = APlayer
→ IsValid(EntityContext.StateMediator)
→ Try Activate Ability by Tag
   AbilityTag = Ability.Weapon.Fire
→ Finish Task
   Succeeded = ReturnValue
```

`Publish Simple`은 유지합니다. 이 이벤트가 없으면 Mediator가 Fire 진입을 알 수 없어 Sprint 취소 같은 상·하체 정책이 실행되지 않습니다.

## 5. Weapon Ability의 EntityContext 캐싱

```text
AbilitySystemComponent
→ UAstraWeaponGameplayAbilityBase.OnAvatarSet 호출
→ ActorInfo.AvatarActor 확인
→ IEntityContextProvider 구현 여부 확인
→ AvatarActor.GetContext
→ CachedEntityContext 저장
```

Ability가 자기 자신에게 `OnAvatarSet`을 호출하는 것이 아닙니다. ASC가 Ability 지급 또는 Avatar 갱신 과정에서 호출합니다. `OnRemoveAbility`에서는 `CachedEntityContext`를 비웁니다.

Weapon Ability는 `CachedEntityContext.WeaponComp`와 `CachedEntityContext.PlayerCameraManager`를 사용하며 구체 PlayerController를 직접 탐색하지 않습니다. 상세 생명주기는 [[PlayerCameraManager 참조 구성]]을 참고합니다.

## 6. GA_WeaponFire 단발 흐름

```text
Event ActivateAbility
→ ResolveWeaponComponent
→ 실패: EndAbility(Cancelled = true)
→ CanFireWeapon
→ 실패: EndAbility(Cancelled = true)
→ CommitAbility
→ 실패: EndAbility(Cancelled = true)
→ FireEquippedWeapon
→ 성공 여부 처리
→ EndAbility(Cancelled = false)

Event CancelAbility
→ 실행 중 Montage/Timer 정리
→ Parent CancelAbility 또는 EndAbility(Cancelled = true)
```

## 7. Draw/Holster Montage

### Draw

```text
STT_WeaponDrawing.EnterState
→ WeaponComp.EnterDrawingCarryState
→ WeaponComp.ExecuteDrawWeapon
→ Montage Notify: Weapon_AttachToHand
→ Montage 종료 Delegate
→ 성공: FinishTask(true)
→ 실패/중단: CancelDrawWeapon + FinishTask(false)
```

### Holster

```text
STT_WeaponHolstering.EnterState
→ WeaponComp.EnterHolsteringCarryState
→ WeaponComp.ExecuteHolsterWeapon
→ Montage Notify: Weapon_AttachToHolster
→ Montage 종료 Delegate
→ FinishTask

ExitState 중 진행 중이면
→ CancelHolsterWeapon
```

## 8. Aim Offset

```text
Event Blueprint Update Animation
→ IsValid(Character)
→ BaseAimRotation
→ ActorRotation
→ Normalized Delta Rotator
→ Break Rotator.Pitch
→ Clamp(-70, 70)
→ SelectFloat(Target = Clamp값 if AimActive else 0)
→ FInterpTo(Current = AimPitch, Target, DeltaTime, AimInterpSpeed)
→ Set AimPitch
```

AnimGraph:

```text
Pistol Idle Base Pose
→ UpperBody Slot
→ AO_Pistol(X = AimYaw, Y = AimPitch)
→ Weapon Upper Pose 선택
→ Layered Blend Per Bone
→ Control Rig / IK
→ Output Pose
```

Fire Montage가 조준 Pitch를 유지하려면 `UpperBody Slot`이 `AO_Pistol`보다 앞에 있어야 합니다. Slot을 Aim Offset 뒤에 두면 발사 시 팔과 어깨가 고정 포즈로 돌아갈 수 있습니다.

## 9. Reload 연결

```text
STC_CanReload
→ Ready, 장착 무기, 탄창 미충전, 차단 상태 없음 검사
→ STT_WeaponReload.EnterState
→ Publish Simple(Event.State.Upper.ReloadEntered)
→ Try Activate Ability by Tag(Ability.Weapon.Reload)
→ GA_WeaponReload
→ WeaponComp.ExecuteReloadWeapon
→ Reload Montage
→ Weapon_ReloadCommit Notify
→ 탄약 반영
→ Montage 완료: FinishTask(true) / 취소: CancelReloadWeapon
→ Ready 복귀
```

빈 탄창 Fire 입력은 Blueprint에서 별도 탄약 분기를 복제하지 않고 `RequestWeaponFire`를 호출합니다. C++가 Fire 또는 Reload 입력 태그를 선택합니다.

---

## 상위 페이지

- [[08_기술 위키_공통 가이드]]

## 관련

- [[Unreal 지식 목록]]
- [[03_시스템 설계]]
