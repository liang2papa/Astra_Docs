# Astra 무기·전투 이슈 가이드

## 설계 의도

일정의 상태 추적과 기술 상세 분리

## 기록 범위

- 기존 남은 이슈·해결된 이슈의 기술 내용 이관
- 미해결 항목은 해결법 확정 문서가 아닌 현재 상태·검증 절차
- 현재 확인일: 2026-09-08
- 이관일: 2026-09-03
- 이번에 확인한 UI 대화·일부 C++ 이외의 전체 구현·PIE 재검증은 미수행
- 원인·해결 방법이 기록되지 않은 부분은 임의 보완하지 않음

## 남은 작업

### WPN-002. Fire/Reload 취소와 상태 충돌 회귀 테스트

- 상태: 회귀 검증 대기

#### 현재 기록·검증 범위

- Sprint, Evade, Jump, Hanging, 피격, 사망 시 Fire/Reload/Draw/Holster가 올바르게 취소되는지 검증한다.
- 발사 종료 후 자동으로 Sprint로 복귀하면 안 된다.

### WPN-003. 연사·반동·HUD 피드백

- 상태: 반동 코드 반영 · 실제 탄 퍼짐 연동 대기

#### 현재 기록·검증 범위

- 발사 확정 시 반동 누적, `PlayerCameraManager` 적용, Fire Montage 종료 후 회복 코드는 반영되었다.
- 단발 Commit과 Continuous 공격 Tick 성공 시 반동이 누적되며, 실패한 공격 요청은 반동을 만들지 않는다.
- 무기별 DataAsset에 1발당 반동 범위·누적 제한·`RecoilRecoveryDuration`을 설정하고 PIE에서 체감과 회복을 검증해야 한다. 기본 회복 시간은 0이므로 미설정 시 즉시 복구된다.
- 이동 사격은 조준점 확산·탄도 오차, 정지 사격은 카메라 반동을 중심으로 분리한다.
- 조준·이동·외부 보정을 반영한 명중률과 확산 계산 함수는 구현됐지만 실제 `ShotDirection`과 HUD 조준점에는 아직 연결하지 않았다.
- 현재 반동 Property는 `RecoilPitchPerShotMin/Max`, `RecoilYawPerShotMin/Max`, `RecoilPitchAccumulationLimit`, `RecoilYawAccumulationLimit`를 사용한다.
- 공중 조준 `80`, 공중 미조준 `0` 데이터는 확정·추가됐지만 현재 명중률 함수는 이동 속도만 검사하므로 Airborne 우선 분기가 아직 필요하다.
- HUD Crosshair의 상태·반동 기반 Scale 갱신 C++는 구현됐지만 실제 명중률과 거리별 확산 반경에는 아직 연결하지 않았다.
- 카메라 조작은 반동 회복을 막지 않으며, 회복 중 새 발사가 성공하면 남은 반동에 새 반동을 누적한 뒤 회복을 다시 시작한다.

### WPN-010. 지속 공격 종료와 Effect 수명 회귀 검증

- 상태: 종료 경로 회귀 검증 대기

#### 현재 기록·검증 범위

- `AttackInstanceId`를 유지하며 Continuous Tick과 Beam Effect Start/Update/Stop이 같은 공격에 연결되는지 확인한다.
- 입력 해제뿐 아니라 탄약 부족, Holster, Unequip, Stun, Death와 Ability Cancel에서도 동일한 종료 경로를 타야 한다.

### WPN-011. RapidFire 반복 발사

- 상태: 미구현

#### 현재 기록·검증 범위

- 현재 일반 총기 발사는 `SingleShot`만 완성되어 있고 `RapidFire`는 enum과 데이터만 존재한다.
- 입력 유지, AttackCooldown 이후 다음 발사 요청, 입력 해제 중지와 빈 탄창 Reload 전환을 구현해야 한다.
- Sprint, Evade, Jump 취소와 Aim Hold 중 연발 유지, 발사별 탄약·반동·HUD 갱신을 검증한다.

### WPN-004. 총구 화염·사운드 동시 재생 제한

- 상태: 연결·튜닝 대기

#### 현재 기록·검증 범위

- Muzzle VFX와 Fire Sound를 Anim Notify 또는 발사 확정 시점에 연결한다.
- 동시 발사음이 과도하게 겹치지 않도록 Sound Concurrency를 설정한다.

### WPN-005. Aim Hold/Toggle과 카메라 확대

- 상태: 통합 검증 대기

#### 현재 기록·검증 범위

- 무기 옵션에 따라 Hold는 짧은 확대, Toggle은 큰 확대를 지원해야 한다.
- Aim 입력, 카메라 FOV, 자동 수납 타이머 초기화, Sprint 해제를 하나의 흐름으로 검증한다.

### WPN-006. HUD 탄약 표시

- 상태: 기본 표시 완료 · Available/Reservation 표현 대기

#### 현재 기록·검증 범위

- 기본 HUD Widget·DataAsset·Component·Controller 중계 구현
- 무기 아이콘·현재/최대 탄약·발사 시 감소·초기값 표시는 PlayerHUD 작업에서 사용자 완료 확인
- 현재 C++ DisplayAmmo 입력: CurrentAmmo / MaxLoadedAmmo / AvailableAmmo / bUsesAmmo
- WeaponInstance 등록 탄수와 Inventory Available 탄약을 원본으로 사용
- Available 탄약과 Preferred/Reserved 표현의 Blueprint 연결·검증 잔여
- 기본 표시 완료를 탄약 Reservation UI 전체 완료로 확대하지 않음
- [HUD 값과 아이콘 표시](../Unreal/UMG/HUD%20%EA%B0%92%EA%B3%BC%20%EC%95%84%EC%9D%B4%EC%BD%98%20%ED%91%9C%EC%8B%9C.md)

### WPN-007. Projectile 충돌 피드백

- 상태: 연결·검증 대기

#### 현재 기록·검증 범위

- 표면별 분기는 보류하되 기본 충돌 VFX와 Sound는 `AmmunitionDataAsset`에서 지정한다.
- `AmmunitionDataAsset`이 ProjectileActorClass·비행·충돌·Impact 설정을 함께 소유한다.
- 별도 ProjectileDataAsset·ProjectileID는 사용하지 않는다.

## 해결 기록

### WPN-001. 빈 탄창 공격 요청의 자동 재장전

- 상태: 해결 완료
- 발사 요청 시 탄약이 없고 재장전할 수 있으면 `Input.Ability.Reload`로 라우팅한다.
- Holstered 상태에서는 Draw 완료 후 Reload를 Pending Action으로 실행한다.
- 재장전 불가능한 무기와 최대 탄창 상태는 Reload로 잘못 진입하지 않는다.
- 완료 확인: 2026-08-28 사용자 확인

### WPN-008. Draw 정렬 대기 Blueprint 연결

- 상태: 해결 완료
- `Abs Delta Yaw <= WeaponDrawAlignmentToleranceYaw` 순서로 비교하도록 뒤집혀 있던 입력을 수정했다.
- 허용 오차 안에서 `NotifyWeaponDrawAlignmentCompleted`가 호출되고 무기 부착과 Pending Action이 이어지는 것을 확인했다.

### WPN-009. 총열 시각 보정과 초근접 제한

- 상태: 해결 완료
- Pitch/Yaw 최대 보정값을 각각 `30°`로 설정하고 Yaw 보간 Target 연결 오류를 수정했다.
- 평사·상향·하향 조준에서 논리 발사 방향과 총구 방향이 일치하며, 보정 해제 시 값이 0으로 복귀한다.
- Fire Montage 중 총구선의 일시적 이탈은 시각 반동이며 Projectile 방향은 유지되고 Montage 종료 후 다시 정렬된다.

### 카메라 Pitch와 Projectile 방향

- 상태: 해결 완료
- 비무장·권총 상태별 Camera Pitch Min/Max 적용 확인
- 카메라 Ray 목표점과 Muzzle 위치를 분리해 조준점 기준 발사 방향 계산
- 현재 발사체 생성 주체: ProjectileFactorySubsystem

### 잘못된 Fire Montage 연결

- 상태: 해결 완료
- 원인: Weapon DataAsset의 잘못된 Fire Montage 참조
- 해결: 실제 Fire Commit Notify가 있는 Montage 연결
- 확인 순서: 현재 무기 DataAsset → 연결 Montage → Notify 이름·시점 → 발사 요청 → Factory 결과
- 근거: [2026-08-17 기록](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%EC%9D%BC%EC%9D%BC%20%EA%B8%B0%EB%A1%9D/2026/2026-08-17.md), [2026-08-25 기록](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%EC%9D%BC%EC%9D%BC%20%EA%B8%B0%EB%A1%9D/2026/2026-08-25.md)

### 기본 HUD 표시

- 상태: 기본 표시 해결 완료
- 무기 아이콘·현재/최대 탄약·발사 갱신·초기 동기화·크로스헤어 표시
- 해상도 전체 회귀·Available 탄약·실제 탄 퍼짐과 Scale 연동은 별도
- [HUD 표시 가이드](../Unreal/UMG/HUD%20%EA%B0%92%EA%B3%BC%20%EC%95%84%EC%9D%B4%EC%BD%98%20%ED%91%9C%EC%8B%9C.md)
## 점검 순서

```text
입력 요청 → StateTree 사전 조건 → GA 활성화 재검사
→ Montage / Commit Notify → WeaponComponent 실행
→ ProjectileFactorySubsystem 생성 결과 → 충돌·피드백

실패·취소 → 공통 종료 → Timer / Effect / Montage 정리
→ 상태·탄약·HUD 최종값 확인
```

## 상태 목록·관련 문서

- [남은 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%EB%82%A8%EC%9D%80%20%EC%9D%B4%EC%8A%88.md)
- [해결된 이슈](../../01_%EC%9D%BC%EC%A0%95%20%EA%B4%80%EB%A6%AC/%ED%95%B4%EA%B2%B0%EB%90%9C%20%EC%9D%B4%EC%8A%88.md)
- [Troubleshooting 목록](Troubleshooting%20%EB%AA%A9%EB%A1%9D.md)
- [정리 근거](../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
