# 인간형 AnimInstance 갱신 구조

> 갱신: 2026-10-23. C++ 데이터·생명주기·행동 몽타주·무기 부착·자세별 몸 Yaw 설정 소유권 관리
> Editor 부모·핀·소켓·무기별 포즈·회귀표는 [무기 애니메이션](../04_아트%20및%20애니메이션/애니메이션%20설계/무기.md)으로 통합

## 설계 의도

AnimGraph는 표현, UAstraHumanoidAnimInstance는 입력 수집·보간·구독과 ABP 기본값의 행동 몽타주 조회 담당. 게임플레이 상태·무기 정책은 기존 실행부 소유

## 구현과 소유권

| 소스 — Source/Astra_SoulLink 기준 | 책임 |
|---|---|
| Entity/Animation/AstraHumanoidAnimInstance.h/.cpp | AnimationData·ActionAnimations·WeaponAttachmentTransforms·WeaponFacingYawOffsets, Entity 연결·해제·프레임 갱신·설정 조회 |
| Entity/EntityCharacter.cpp | InitClass의 컴포넌트 준비 후 인간형 AnimInstance 직접 초기화, EndPlay 구독 해제, 플레이어 FaceRotation의 몸 Yaw 보정 |
| Entity/NPCCharacter.cpp / NPCAIController.cpp | Focus 회전과 NPC 발사 허용 검사의 동일한 몸 Yaw 보정 기준, 기존 컨트롤러 갱신 경로 사용 |
| Entity/Combat/Weapon/Handlers/CarryHandler.h/.cpp | 현재 ABP의 ItemID별 Transform으로 실제 손/수납 부착·변경 내부 알림 |
| Entity/Combat/Weapon/WeaponComponent.h/.cpp | 부착 알림 Facade 중계, 장착·해제·Carry 공개 이벤트 |
| BP_PlayerCharacter / BP_NPCCharacter | 기존 Cast→InitABPClass 체인 제거 대상. AnimInstance 초기화 직접 호출 없음 |
| 각 캐릭터 ABP | 기본값에 FireMontages·ReloadMontages·MantleMontage·DeathMontages·WeaponAttachmentTransforms·WeaponFacingYawOffsets·AimPitchInterpSpeed 지정, AnimGraph에서 AnimationData 표현 |
| AttackSequenceHandler / ReloadHandler / EntityLocomotionComponent | 현재 AnimInstance의 설정 조회 후 기존 발사·재장전·Mantle 실행·완료·중단 복구 |

기존 BP 변수 충돌을 피하기 위해 새 데이터는 AnimationData 구조체로 제공. 부모 교체만으로 기존 Getter 핀 자동 이관을 가정하지 않음. 빈 BlueprintInitializeAnimation 핀과 사용자 BP Update 연결 해제 유지

## 데이터 갱신

| 값 | 방식 |
|---|---|
| CurrentWeaponType / WeaponCarryState | 장착·해제·Carry 이벤트로 dirty 표시, 다음 NativeUpdateAnimation에서 한 번에 스냅샷 |
| 실제 손 부착 | OnWeaponAttachmentChanged로 동일 dirty 표시. CarryState 이벤트만으로 Draw/Holster 내부 부착 시점 대체 불가 |
| bWeaponUpperBodyActive | 데이터 존재 AND 실제 손 부착 |
| bEnableWeaponAimOffset | 상체 활성 AND Carry Ready |
| Velocity / GroundSpeed / Direction / bShouldMove | NativeUpdateAnimation에서 Movement 입력 계산 |
| bIsFalling / bIsLedgeGrabbed | Movement·Locomotion 현재 값 |
| AimPitch / AimYaw | 상대 BaseAimRotation Pitch 보간, Yaw=0 유지 |
| VisualAimCorrectionRotation | Weapon 보정값 → Roll=-VisualPitch, Yaw=VisualYaw. 추가 가산 없음 |

기존 그래프에서 이관한 명명 상수: GroundSpeed 임계 0.01, 이동 방향 제한 45도, AimPitch 제한 80도. 방향 제한 여부는 CharacterMovement.bOrientRotationToMovement 현재 값 사용. AimPitchInterpSpeed는 ABP 기본값. 기존 AimInterpSpeed 값을 사용자 이전, 미확인 값의 임의 생성 없음. AnimGraph의 AimYaw=0 유지, 몸 Yaw 보정은 아래 별도 설정 사용

## 행동 몽타주 설정

- Class Defaults → Animation | Actions → ActionAnimations
- FireMontages: EWeaponType별 UAnimMontage 참조. 현재 ABP의 스켈레톤용 에셋만 등록
- ReloadMontages: EWeaponType별 재장전 몽타주. ReloadHandler에서 현재 무기 종류로 조회
- MantleMontage: 현재 캐릭터의 올라타기 몽타주
- DeathMontages: 사망 몽타주 배열. 빈 배열·None 항목은 Error. 유효 배열에서 사망 시작 시 하나 무작위 선택, 같은 사망의 재요청은 재선택 없음. 반복 사망 간 같은 항목 선택 허용
- EntityCharacter의 단일 DeathMontage 제거. 현재 ABP의 SelectDeathMontage 결과를 ActiveDeathMontage에 보관 후 기존 재생·블렌드아웃·제거 처리
- WeaponDataAsset.FireMontage·ReloadMontage 제거. Equip·Holster 몽타주는 이번 이전 범위 밖
- 사용자 요청으로 미사용 DryFireMontage·DryFireSound 제거. ABP에 대체 설정 추가 없음
- GetFireMontage / GetReloadMontage / GetMantleMontage: 설정 조회만 수행. 추가 프레임 순회·동기 에셋 로딩 없음
- AttackSequenceHandler: 실제 WeaponData.WeaponType으로 조회. Weapon_FireCommit·탄약·쿨다운·종료 Delegate 기존 처리 유지
- ReloadHandler: 정상 몽타주 종료 시에만 탄약 확정. 취소·중단 시 미확정과 이전 콜백 세대 검사 유지
- AimPitchInterpSpeed는 기존 ABP AimInterpSpeed의 조준 Pitch 보간 설정. WeaponDataAsset.VisualAimCorrectionInterpSpeed는 총열 시각 보정용 설정으로 기존 위치·AimHandler 소비 경로 유지
- ExecuteMantle(): Task의 몽타주 인수 제거. LocomotionComponent에서 현재 ABP 설정 조회 후 목표점·Warp·이동 복구 처리
- 부모 AnimInstance 또는 필수 몽타주 누락 시 실행부 Error 로그와 실패 결과. 이전 무기 DA·Task 설정으로 우회 없음

## 무기 부착 설정

- Class Defaults → Animation | Weapon Attachment → WeaponAttachmentTransforms
- 키: WeaponDataAsset.ItemID와 같은 FName. WeaponType은 애니메이션 선택 분류이며 부착 설정 키와 별개
- 값: FAstraWeaponAttachmentTransforms의 EquippedMeshTransform·HolsteredMeshTransform. 소켓 기준 상대 위치·회전·크기
- 같은 무기 종류의 서로 다른 모델에도 개별 보정 가능. 같은 무기를 사용하는 캐릭터는 각 ABP에 자기 체형용 값 지정
- 무기 DA의 두 Transform 제거. SocketName·WeaponStaticMesh·총열 시각 보정 수치는 기존 무기 DA 유지
- 최초 Bind·손 부착 유지 교체: 새 ItemID 설정을 기존 실행 취소 전에 검사. 누락 시 기존 무기 취소·새 무기 연결 없이 실패
- Draw·Ready·Holster 부착: 현재 ABP와 현재 WeaponInstance의 ItemID 조회. 성공 시에만 실제 부착 상태 알림
- 빈 ItemID·누락 항목은 Error와 부착 실패. 항등 Transform이 필요할 때도 명시적 항목 등록
- 설정 조회는 무기 연결·부착 시 수행. 신규 Tick·ABP 런타임 재생성 없음
- 기존 단일 사망 설정·무기 DA Transform의 소유 객체가 변경되어 자동 Redirect 이전 없음. 사용자 값 기록 후 각 ABP 기본값에 직접 이전

## 무기 자세별 몸 Yaw 보정

- Class Defaults → Animation | Weapon Facing → WeaponFacingYawOffsets
- 키: EWeaponType. 값: 현재 캐릭터의 해당 무기 자세에 필요한 몸 Yaw 보정 각도. 미등록 종류는 0도, 튜닝 전 임의 각도 지정 없음
- EntityCharacter.GetWeaponFacingYawOffset: 실제 손 부착과 현재 무기 데이터를 직접 조회. 다음 AnimationData 스냅샷 대기 없음. 수납·비무장·사망 시 0도
- 플레이어 FaceRotation: 전달받은 Control Yaw 복사본에 보정 후 기존 엔진 회전 적용. 현재 Actor Yaw 누적·ControlRotation 수정·카메라 조준선 수정 없음
- NPC Focus 회전: 타겟 방향에 같은 보정 적용. IsFacingCombatTarget의 허용 오차도 같은 목표 몸 방향 기준, 자세 보정 때문에 공격이 차단되는 문제 방지
- NPCAIController.UpdateControlRotation: Focus Actor가 있는 NPC의 몸 회전은 기존 컨트롤러 갱신에서 한 번 적용. Control Yaw와 몸 Yaw가 같은 프레임에도 손 부착·설정 변경 반영. Focus Actor가 없는 이동 방향 회전은 기존 경로 유지
- 수납 완료 시 기존 이동 방향 회전 정책 복귀. 보정값의 역회전 별도 실행 없음. 무기 교체 시 현재 무기의 종류 재조회
- 신규 Tick·Timer·GA·Task 없음. 고정 좌우 자세 오차용 설정이며 거리별 시차·상하 조준 오차 전체 해결과 구분

## 총구·조준점 정렬

- 과거 NPC 자세 복구 기록: 유효 Aim Solution에서도 추가 척추 보정 비활성. 10/22 성공 분기를 단순 오류로 해석한 자동 보정 재활성화 철회
- EntityCharacter.Tick에서 논리 Aim Solution은 계속 갱신. 실패·타겟 없음 시 Clear. 총구 오차의 추가 척추 누적은 Reset 유지, 기존 AimOffset 표현 사용
- 카메라 레이의 방향 유지, 충돌 시작점은 카메라 레이 위에서 Owner·Muzzle의 깊이 이상으로 전진. EffectiveRange는 전진한 시작점부터 적용. 카메라 거리 때문에 조준 끝점이 총구 뒤에 생기는 경로 제거
- 기존 AttackCommitHandler의 실제 Muzzle 위치→AimTarget 방향 재계산 유지. 시각 소켓 +X를 발사 방향으로 강제하지 않음
- 시각 보정의 현재 총열 방향은 Muzzle 소켓 월드 +X 축. 새 메시의 소켓 위치·회전·AO·척추 보정 좌표와의 실제 일치 여부는 Editor·PIE 확인 대상
- astra.Weapon.DebugAim=1: 전진한 카메라 조준선 파랑, Muzzle +X 노랑, Muzzle→AimTarget 청록 표시. 기본값 0, 2에서 목표점 문자 추가
- astra.Weapon.DebugAimAlpha 기본 0.2. 파랑의 알파 255 강제 제거, 선 두께 0·목표점 작은 와이어 구로 변경
- 비교 근거: 로컬 LyraGameplayAbility_RangedWeapon.GetTargetingTransform의 카메라 레이 시작점 전진. 같은 결과를 강제하는 무기·팔 회전 코드의 복제 없음

## 초기화·해제·스레드 경계

```text
AEntityCharacter.InitClass
→ Locomotion·ASC·Weapon 준비, Context·Ability 연결
→ 메시의 현재 UAstraHumanoidAnimInstance.InitializeForEntity(this)
→ 기존 구독 해제 → 소유 Entity/Movement/Locomotion/Weapon 확인
→ 장착·해제·Carry·실제 부착 이벤트 구독 → 현재 무기 1회 스냅샷
→ Base·Upper StateTree 시작

초기화된 Entity의 메시·ABP 재생성
→ NativeInitializeAnimation → 같은 초기화 경로 재연결
프리뷰·Entity 초기화 전 → 연결 대기, 임의 Tick 재시도 없음

무기 이벤트 → dirty 표시
NativeUpdateAnimation(게임 스레드)
→ dirty면 무기 스냅샷 갱신 → 연속 입력 수집·보간 → AnimationData
→ AnimGraph 표현

무효 소유 참조 / Entity.EndPlay / NativeUninitializeAnimation
→ 구독·참조 해제 → 비무장·비활성 데이터 초기화
재초기화 → 기존 구독 해제 후 다시 연결
```

현재 NativeThreadSafeUpdateAnimation·Proxy 워커 스레드 이관 없음. 향후 병렬화 시 게임 스레드 준비 스냅샷·수학 계산만 전달하며 Actor·Weapon·ASC 일반 Getter를 워커에서 직접 호출하는 구조로 가정하지 않음. 추가 Actor/Component Tick 없음

## 반영·검증 기록

- 10/19 C++ 반영. 초기 Development Editor / Win64 빌드 로그: Saved/BuildLogs/2026-10-19-AnimInstance-DevelopmentEditor.log. 이후 변경 전체의 빌드·실행 성공 근거와 구분
- 손에 든 A→B 교체 시 새 손 소켓·EquippedMeshTransform 적용·Ready 유지, Player의 불필요 Draw 재요청 생략. 초기/비무장은 기존 수납·Draw 정책
- 사용자 BP Update 연결 해제와 AO 참조 변경 후 왼팔 해결 확인. PT-00-3 패스 유지
- 실제 연결·취소/교체·재초기화 검증은 무기 애니메이션 문서의 통합 표 사용. 무기 1 탄창 0→2번 발사 잔여는 WPN-015, 재장전은 사용자 완료로 구분
- 비교 자료: 로컬 Lyra Source/LyraGame/Animation/LyraAnimInstance.h/.cpp. 모든 Lyra 값이 C++에 있다는 해석은 제외
- 10/22 C++ 반영: 발사·재장전·Mantle ABP 설정, 미사용 DryFire 설정 제거, Entity 공통 초기화와 엔진 재초기화 연결. 최신 사용자 빌드·Editor 연결·PIE 대기
- 10/22 후속: 사망 몽타주 배열·무기 ItemID별 부착 설정 이전. 시각 보정 재활성화 후 자세 회귀 보고·과거 의도 확인, 추가 보정 자동 활성화 철회. 레이 시작점 전진·디버그 가독성 수정 반영, 빌드·실제 정렬 확인 대기
- 10/23 C++ 반영: ABP WeaponType별 몸 Yaw 보정, 플레이어 FaceRotation·NPC Focus 회전·발사 허용 기준 연결. 사용자 머리 들기 처리 완료·기본 자세 총기 좌우 정렬 정상 확인. 상하 조준 시 총기 위치·회전 조정 검토, 전체 무기·캐릭터 조합의 실행 결과 미확인

## 관련 문서

- [무기 애니메이션 연결·검증](../04_아트%20및%20애니메이션/애니메이션%20설계/무기.md)
- [WeaponComponent 기능 분리](WeaponComponent%20기능%20분리.md)
- [현재 작업 상태](../01_일정%20관리/현재%20작업%20상태.md)
