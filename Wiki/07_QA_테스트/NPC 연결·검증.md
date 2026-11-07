# NPC 연결·검증

## 조종·실행 참조 경계 회귀 — 2026-10-28

사용자 승인한 공통 입력/사건 경계 C++ 반영. **추가 BP·Task·GA·DA 연결 없음.** Controller가 C++ Possess/UnPossess에서 Native 조회와 기존 사건을 연결·해제한다. 기존 Base/Upper/Brain 공개 API와 우선순위는 유지한다. [코드 책임·수명](../03_시스템%20설계/시스템%20아키텍처.md#반영한-입력사건-수명) 기준.

```text
AI Possess → Entity 초기화 → 입력/명중률 조회 제공 + Mediator/Health/Weapon/Skill 구독 → Brain 시작
실행 → 조회 당시 제공된 대상/관측/보정 값 사용 → 기존 GA/Facade/Handler 실행
근접 접근 시작 → Handler 요청 → Weapon Facade 사건 → AI 경로 Keep 해제 → 같은 호출에서 접근 이동
피격·착지 → 기존 Mediator 사실 → AI 구독자 판단 (Character 중계 없음)
HP 0 → Health 사실 → AI Brain/공격/경로/타겟 정리 + 기존 Entity 사망 경로
UnPossess/EndPlay → 자기 조회 바인딩·사건 구독 해제 → 다음 조종 제공자 연결
미조종/제공자 없음 → 빈 입력, 이전 타겟/기억 조회 없음
```

Health 사망 사실 구독으로 Brain 정리는 이전 사망 몽타주 진입보다 이른 HP 0 시점이다. Base Death Critical 전이·GA/몽타주·전리품·보스 자기 소환물 제거는 기존 경로 유지. 일반 피격 행동 중단 여부는 기존 DA 설정을 유지한다.

| 조건 | 사용자 빌드 후 기대 결과 |
|---|---|
| 일반 NPC 정지/이동 사격 | 기존 몸 정렬·사선·라이플 산포식 유지 |
| 보스 충전 중 가림/Chase 전환 | 충전 유지 조건은 기존 정책, 마지막 관측 위치 발사. 숨은 대상 현재 위치 추적 없음 |
| 좀비 Chase→근접·접근 중 횡이동 | 이동 인계 때 임의 정지 없음, 접근 중 매 Tick 추적·거리/시간 한도·끝 공격 모션 유지 |
| 근접 이동 인수 도중 취소/사망 | 취소 후 접근 속도를 다시 적용하지 않음 |
| 가려진 좀비 타겟 / 기억 만료 | 기억 좌표 사용, 기억 소실 시 기존 취소 정책 |
| Player 근접 | 기존 카메라 조준점 후보 선택·무대상 제자리 공격·진영/소켓 타격 유지 |
| 착지/피격 | 현재 Brain 이동 의도 재요청·피격 위치 기억 정상. 불필요한 DamageLook/일반 공격 취소 재발 없음 |
| 무조종·AI 재소유·Player로 교체 | 이전 제공자의 대상/명중률·구독 잔류 없음. 향후 빙의 전체 기능 완료를 뜻하지 않음 |
| HP 0 / 공격·채널링 중 사망 | AI 즉시 정리, Base Death/몽타주·전리품 정상. 자기 소환물만 제거 |
| SkillComponent 없는 일반 NPC/좀비 | 피격·착지·사망·근접 인계 구독 정상. 선택적 스킬 유무와 독립 |

13개 기존 Source diff/선언/정의/호출·신규 계약 헤더·구독/해제·값/순서 모델 20조건 정적 확인. **이번 소스의 빌드·UHT·Editor·Blueprint/StateTree Compile·물리/애니메이션/PIE는 미수행.** 전체 `Astra_SoulLinkEditor / Win64 / Development` 빌드와 재시작 후 확인한다.

## 최신 확인 기준 — 2026-10-26

원거리 모서리/총구 끼임 후속: [발사 공간·총구 끼임 설계](../03_시스템%20설계/NPC%20행동.md#원거리-발사-공간총구-끼임--2026-10-26). NPC 반경=max(몸 Capsule 반경, 탄 CollisionRadius)로 몸통→총구→타겟 Sweep. 막히면 기존 사선 false 전이로 Chase, 사선 확보 전에는 기존 도착 허용 반경에서 멈추지 않도록 0으로 요청. API/Task 추가 없음. 정적 점검만 수행, 빌드·PIE 전.

검증: 모서리에서 중심 선만 보이는 배치·총구가 벽 반대편으로 나온 배치·시작 겹침·충분히 열린 공간·같은 진영 NPC 통과·뒤 벽 차단·추격 중 타겟 정지·충전 중 이동·사망/취소. 기존 NPC Has Clear Firing Line 조건의 Attack→Chase 반전/Chase→Attack 정방향 연결 확인. 일반 연사와 근접 좀비 완료 대기 회귀도 확인.

사용자 제공 STT_NPC_Attack의 실제 핀 연결·오류·수정 후 정상/실패/취소 전체 흐름은 시스템 설계의 [공격 Task Blueprint 구조](../03_시스템%20설계/NPC%20행동.md#공격-task-blueprint-구조--2026-10-26)에서 관리. Tick의 GA 완료 감지는 유지하며 매 Tick GA 재활성화와 구분. 같은 그래프 전체를 다시 요청하기 전에 이 기록 확인. Editor 적용·Compile·연사 PIE는 미확인.

승인된 [같은 진영 총탄 통과·오사 차단](../03_시스템%20설계/NPC%20행동.md#같은-진영-총탄-통과오사-차단--2026-10-26) C++/Config 반영. 전체 빌드 및 Editor 재실행 후 다음 확인:

- Enemy NPC 두 기를 앞뒤 배치: 뒤 NPC의 총탄이 앞 NPC를 통과하고 앞 NPC의 HP/피격 반응은 변화 없음
- 통과한 탄이 뒤의 플레이어/벽에는 정상 충돌, 벽 뒤 플레이어에게 피해 없음
- Allies 진영도 동일 정책, 서로 다른 진영은 정상 피해
- 보스 폭발은 같은 Enemy에 피해 없음, 적대 대상 피해와 벽 차폐 유지
- 발사체 풀 재사용 시 이전 소유 진영의 프로필 잔류 없음, 비행 중 나중에 소환한 같은 진영 좀비도 통과
- DA Faction 미설정(None)·Neutral은 이번 동일 진영 통과 정책에 포함하지 않음

## 근접 좀비 연결 준비 — 2026-10-26

후속 변경: 접근 중 기존 이동 ABP→거리 도달 후 공격 몽타주. 몽타주는 공격 시퀀스+Melee_Hit 창만 사용, Melee_Lunge 불필요. 새 Trace Socket Names 배열에 실제 공격 손/날/방망이 타격부 지정, 맨손/장착 근접 무기로 메시 자동 선택. TraceRadius는 두께+작은 접촉 여유. 전방 사거리 Sweep 폐기, 포즈 확정 후 소켓 궤적만 검사. 새 설정의 빌드·PIE 미수행. 30/60 FPS·사선 내려베기·손 빗맞음·창 밖 비피해·벽·취소/사망·다중 좀비 회귀 확인 필요.

기존 Handler 근접 분기 C++ 반영. 사용자 완료 보고는 좀비 BP·ABP 연결까지이며 StateTree·구체 GA·DA·Montage Notify 연결과 이번 빌드·PIE는 별도. 맨손은 가짜 무기 없이 ASC 비무장 수치, 공격몽타주는 ABP Fire Montages의 None 키 사용. [DA 필드·Notify 창·정상/실패/취소 전체 흐름](../03_시스템%20설계/근접%20공격·소환기%20사전%20점검.md#현재-반영-기존-handler-근접-분기) 참조. 보스 소환기는 이번 근접 기반 반영의 후속.

## 피격 후 Chase 정지 수정 — 2026-10-25

사용자 재현: 보스만 사격 시 추격 중 정지, 플레이어가 사거리 안으로 들어오면 발사. 같은 무기를 든 일반 NPC도 정상 동작하는 공통 구조로 수정 요청. 이번에는 Editor 직접 편집 허용.

### 확인한 원인과 반영

- 제공한 `STT_NPC_ChaseWalk` 그래프는 Enter에서 `RequestChaseMove`, Tick에서 `UpdateChaseMove`, Exit에서 `CancelChaseMove`. 반복 Tick에서 종료하거나 사거리까지 기다렸다 이동을 재시작하는 구조가 아님.
- 제공한 보스 Base 텍스트와 실제 Editor의 보스·일반 NPC Base에서 `DamageLook` Task 인스턴스의 `Considered for Completion=false` 확인. 공유 Blueprint의 Class Defaults는 true이지만 배치 인스턴스는 별도 false. 실제 Blueprint에는 Tick 정상 완료 시 EndDamageLook/FinishTask, Exit 시 EndDamageLook/참조 정리가 이미 존재. 완료 전이가 없는 것으로 추정했던 이전 가설을 수정하고 **완료 판정 참여 누락**으로 진단 구체화.
- 두 Base의 DamageLook Task 완료 판정 참여를 true로 수정. 기존 On State Completed 조건은 현재 BrainState를 검사해 Chase→ChaseWalk, Attack→Idle, Idle→Patrol 선택. ChaseWalk Enter가 이동을 재요청하며 무조건 EndDamageLook에서 이동을 재개하는 코드 추가 없음.
- 공유 `STT_NPC_DamageLook`의 `Should State Change on Reselect=false` 설정. 반복 피격의 동일 상태 재선택으로 Exit가 최신 Pending 방향을 지운 뒤 Enter가 실행되는 경로 방지. 추가 피격 방향은 기존 UpdateDamageLook에서 갱신.
- 두 Base의 Ground→ChaseWalk=Medium, Ground→DamageLook=High로 통일. 사망 Critical 유지. Chase와 피격 명령이 같은 평가에서 경쟁할 때 피격 반응 우선, 순찰 취소 Normal보다 추격 우선.
- 실제 BP_BossCharacter의 Entity Data Asset은 DA_NPC_DemoBoss. 해당 DA의 `Can Interrupt Actions on Damage=true`를 Editor에서 확인 후 앞서 확정한 보스 정책에 맞춰 false로 저장. 일반 NPC DA는 변경 없음.
- C++ `StateMediator.CanInterruptActionsOnDamage()`로 DA·생존·기절 검사를 공통화. HandleDamageTaken과 직접 SendLowerCommand(HitReaction) 모두 정책 검사, Controller의 Remember/Begin/Update도 같은 조회 사용. 사망 명령과 명시적 행동 취소는 별도 유지.
- 반응 활성 중 AI UpdateControlRotation과 타겟 Focus 갱신이 피격 회전을 덮어쓰지 않도록 수정. Brain Attack 진입에서 EndDamageLook을 호출하지 않으며 새 발사는 반응 완료까지 기존 UpdateCombatAttack에서 대기. 반응 중 Brain 추격/패트롤 명령 재발행은 생략하고 완료 시 현재 Brain으로 복귀. 피격 방향 인지와 무기 발사 스타일 분리, 보스 클래스·무기 종류 분기 없음.
- 중앙 `astra.Debug NPC.Combat 1`에 기존 TraceNPCDecision을 사용한 DamageLookStarted/Ended 사건 추가. 신규 콘솔 변수·Source 테스트·검증 스크립트 없음.

```text
Health 피격 사실 → 방향/기억 중계
├─ 중단 정책 false → 현재 이동·회전·충전 유지
└─ 중단 정책 true → Upper 취소 + Lower HitReaction
   → DamageLook Enter → 정책 재검사
      ├─ 반응 불필요 → FinishTask(true) → 현재 Brain 기준 완료 전이
      └─ 반응 시작 → 이동/사격 취소 → 저장 방향으로 회전
         ├─ 추가 피격 → 기존 Task에서 방향 갱신
         └─ 정상 완료 → EndDamageLook → FinishTask(true)
            → Chase: ChaseWalk Enter → RequestChaseMove
            → Attack: Base Idle → 기존 Brain 공격 Task에서 요청 재개
            → Idle: Patrol
강제 이탈/사망 → Exit 정리만 수행, EndDamageLook의 무조건 추격 재개 없음
```

### 검증 범위와 남은 확인

- Editor: 공유 피격 Task Blueprint 컴파일 성공, 두 Base StateTree 컴파일 성공 및 저장. 기존 비활성 디버그 텍스트 Task 경고만 확인. DA false 저장 확인.
- Source: 변경 전 백업 비교·선언/정의/호출·공통 정책 직접 참조 및 공백 검사. 변경 소스 3개, 콘텐츠 4개. 백업 `Saved/CodexChanges/DamageReactionRecovery-20261025-210244`.
- 이번 C++ 전체 빌드·UHT·PIE 미수행. 에디터에 로드된 DLL은 이번 소스 변경 전 버전이므로 에셋 컴파일 성공을 C++ 실행 검증으로 간주하지 않음.
- 빌드 후 PIE: 보스의 추격/충전 중 연속 피격 유지, 반응 허용 일반 NPC의 추격 중 피격 후 이동 재개, 반응 중 Brain Chase↔Attack 변경 후 올바른 복귀, 연속 피격 방향 갱신, 반응 중 사망 우선권 확인. 같은 충전 무기를 든 일반 NPC도 해당 Entity 정책에 따라 동작하는지 확인.
- HP 기준 기절 상태 구현·일반 NPC 이동 사격 정책·보스 스킬은 이번 범위 밖.

## 현재 작업: 로코모션 수치 통일·탄착 경고·폭발 효과 — 2026-10-25

사용자 요청으로 추격/회전 이슈 추적을 종료하고 탄착 경고·바닥 폭발 효과 → 보스 스킬 순서로 진행. 종료 지정은 이번 C++ 변경의 빌드/PIE 성공 증거와 구분한다.

### 로코모션 수치 소유

- 무장 전용 `WeaponFacingRotationRateYaw`와 비무장 회전속도 캐시 제거. Entity DA `RotationSpeed` → ASC → CharacterMovement.RotationRate.Yaw → NPC FaceRotation으로 통일. 무장/수납은 회전 방식만 바꾸며 별도 속도를 적용하지 않음.
- DA Attribute 초기화 후 회전속도 반영, ASC RotationSpeed 변경 Delegate로 GE/복제 변경 반영, EndPlay에서 구독 해제. 별도 Tick·회전 수치 DA 추가 없음. 보스의 현재 720도/초를 낮추려면 Entity DA의 RotationSpeed만 조절.
- 이동속도는 기존 DA BaseMoveSpeed → ASC MoveSpeed → MaxWalkSpeed 경로 유지. C++ 초기/누락 대체값은 별도 튜닝용 DA가 아니며 정상 초기화된 보스의 600을 덮어쓰지 않음.
- ABP의 AimPitchInterpSpeed/MovementSpeedThreshold/MovementDirectionLimit/AimPitchLimit는 포즈 표현용. 몸 Yaw 회전속도와 분리. AimPitchInterpSpeed=0은 상하 조준 즉시 반영, MovementDirectionLimit는 OrientRotationToMovement일 때만 적용.
- 제거한 무장 회전속도 프로퍼티의 BP 노드 참조가 있으면 제거 후 DA 기준으로 연결. 콘텐츠 문자열 검사에서는 참조를 찾지 못했으나 BP Compile 성공 증거는 아님.

### 탄 DA 연결

현재 보스 무기 NonAmmoProjectileData가 가리키는 **동일 탄 DA**에서 아래를 설정한다. 별도 보스 UI/폭발 데이터에 피해 반경을 복사하지 않는다.

| 항목 | 연결 |
|---|---|
| bExplosive | true |
| ExplosionRadius | 실제 피해 반경. 기존 500이면 반경 5m, 지름 10m |
| bAimAtTargetGround | 기존 true 유지 |
| ImpactWarningMaterial | 아래 Deferred Decal 경고 머티리얼 지정. 새 에셋 이름은 사용자 지정 |
| ExplosionVFX | 사용자 확정 `/Game/Free_Spells/VFX_Niagara/NS_Free_Spells_Explosion` |
| ProjectileVFX / TrailVFX | 현재 탄 본체/비행 표현 유지. 바닥 폭발을 여기에 붙이지 않음 |
| DefaultImpactVFX | 기존 접촉 효과용. 같은 폭발을 중복 지정하지 않음 |

코드 연결은 기존 Factory의 발사 성공 뒤 경고 생성 경로 재사용. 경고는 탄 DA ExplosionRadius를 직접 사용하며 비행 곡선 끝점에 고정. 실제 접촉 순간 경고를 먼저 제거하고, 폭발은 CombatEffectSubsystem이 Hit.ImpactPoint/ImpactNormal 기준으로 생성. 바닥/경사면은 표면 +Z 기준, 벽/Pawn 조기 접촉도 실제 충돌 위치에서 재생. 예정 지면으로 효과를 옮기지 않음. 피해 계산은 ProjectileActor의 기존 Explode 경로 유지.

### 경고 머티리얼 구성

원형 테두리 + 반투명 내부를 임시 연결 기준으로 정리. 내부는 FillOpacity=0으로 끌 수 있다. 표현 선택 질문의 최종 응답 전이며 색상/최종 두께는 사용자 지정. 아래 값은 머티리얼 구성용 예시이며 새로운 게임플레이 반경이 아니다.

1. 새 Material: Material Domain=`Deferred Decal`, Blend Mode=`Translucent`.
2. `TextureCoordinate`와 `Constant2Vector(0.5,0.5)` → `Distance`의 A/B. 출력이 D.
3. `SmoothStep(Min=0.49, Max=0.5, Value=D)` → `OneMinus` = OuterMask.
4. ScalarParameter `RingWidth`(예시 0.04). `Subtract(0.49, RingWidth)`/`Subtract(0.5, RingWidth)`를 두 번째 SmoothStep의 Min/Max에 연결, Value=D. 출력 → OneMinus = InnerMask.
5. `Subtract(OuterMask, InnerMask)` → Saturate = RingMask.
6. ScalarParameter `FillOpacity`(예시 0.15) × InnerMask → Add(RingMask) → Saturate → Material Opacity.
7. VectorParameter `WarningColor`(색상 사용자 지정) → Base Color와 Emissive Color. 필요 시 Material Instance에서 색/채움/두께 조절.
8. Compile/Save 후 탄 DA의 ImpactWarningMaterial에 해당 Material 또는 Material Instance 지정. 바닥 메시 Receives Decals 활성화 확인.

```text
TexCoord ─────┐
UV(.5,.5) ────┴→ Distance(D)
                 ├→ SmoothStep(.49,.50) → OneMinus → OuterMask ─┐
                 └→ SmoothStep(.49-W,.50-W) → OneMinus → Inner ─┴→ Subtract → Saturate → Ring
                                                        Inner × FillOpacity ───────────┐
                                                                            Ring ─────┴→ Add → Saturate → Opacity
WarningColor ───────────────────────────────────────────────────────────────────────────→ Base Color / Emissive
```

UV 0.5는 데칼 내 원을 만드는 정규화 좌표. 월드 크기는 C++가 ExplosionRadius로 지정한다. 현재 경고는 고정 범위이며 착탄까지 확대/채움 애니메이션을 추가하지 않음. 선택한 Niagara는 일회성 재생 후 완료하도록 설정하고, 탄 BP의 Hit에서 같은 폭발을 추가로 Spawn하지 않는다. VFX의 아트 크기와 피해 반경은 구분하며 실제 피해/경고 반경은 탄 DA가 소유한다.

### 정상·실패·취소 흐름

```text
충전 시작 → 기존 머즐 구체 성장 (바닥 경고 없음)
  → 발사 Commit 성공 → Factory 활성화 → 고정 예정 지점 경고
    ├─ 바닥 착탄 → 경고 제거 → 폭발 피해·해당 표면 ExplosionVFX → 탄 풀 회수
    ├─ Pawn/벽 조기 충돌 → 경고 제거 → 폭발 피해·실제 충돌점 ExplosionVFX → 탄 풀 회수
    └─ 수명/직접 제거/월드 종료 → 경고 정리, 접촉 폭발을 새로 재생하지 않음
충전 취소/발사 실패 → 경고/착탄 폭발 없음
탄 풀 회수 → 이미 월드에 생성한 일회성 폭발 Niagara는 독립 재생 후 종료
```

경고/폭발 에셋 미지정·생성 실패는 시각 효과만 생략하며 기존 피해/탄 정리를 막지 않음. 두 발의 AttackInstanceId별 경고 독립, 첫 발 조기 충돌이 다른 경고를 지우지 않는지 확인. 이번 C++ 정적 확인까지이며 빌드·UHT·Material/BP Compile·PIE 미수행. Editor 에셋 지정도 사용자 작업. 이번 추가 검증 스크립트/Source 테스트 파일 없음.

다음 보스 스킬은 [프로토타입 계획](../01_일정%20관리/2026년%2012월%20프로토타입%20제작%20계획.md)의 소환기·전멸기를 기준으로 설계. 패턴 확률/간격/겹침, 전멸기 예고 시간/피해 간격·계수 등 미정 수치는 확정값으로 만들지 않음. 이번 탄착 경고를 전멸기의 빨간 두 원 기믹과 혼동하지 않음.

## 보스 무소모 유탄 — 2026-10-24

사용자 확인: 일반 NPC 공격 정상, 보스 시작 무기 생성·추적·유탄 발사·포물선 확인. 기존 WeaponCannotAttack은 참조 연결·중력 설정 후 발사 확인으로 진단 종결. 후속 목표: 플레이어 발밑 지면 조준, Pawn·지형 접촉 즉시 폭발. 지면 조준 C++ 반영, 사용자 빌드·설정·PIE 대기

### 무기 DA 연결

대상: `/Game/Astra/DataAsset/Item/Equipment/Weapon/DemoBossSpecialWeapon/DA_DemoBoss_BaseGR`

1. 사용자가 전체 Development Editor 빌드 후 Editor 재시작. Codex의 추가 빌드·UHT·헤드리스 Editor 실행 없음
2. 기존 `AttackKind=Ranged`, `DeliveryType=PhysicalProjectile`, `ShootType=SingleShot`, `Uses Ammo=false` 유지
3. `Weapon | Projectile → Non Ammo Projectile Data`에 기존 `/Game/Astra/DataAsset/Item/Ammunition/DA_BaseGR_Grenade` 지정
4. 사용자 제공 BaseDamage=80, AttackCooldown=1, FirstFireDelayTime=0.3, EffectiveRange=10000, AutoHolsterDelay=0 유지. FirstFireDelayTime은 기존 발사 경로 설정이며 실제 Commit 시점은 Montage Notify 기준
5. AmmoTypeTag·MaxLoadedAmmo·AmmoPerShot·ReloadTime은 무소모 실행에서 탄창·재장전 조건으로 사용하지 않음. `Default Ammunition Data`로 무소모 발사체를 등록하는 추가 BP 노드 없음
6. 보스 EntityDataAsset의 `Can Reload=false` 설정. 기존 일반 NPC의 CanReload·무제한 예비 탄약 설정 유지

### 유탄 DA 연결

기존 `DA_BaseGR_Grenade` 수정. 새 탄약 Stack·Inventory·등록 탄수 생성 불필요

| 항목 | 설정 |
|---|---|
| Projectile Actor Class | 네이티브 `ProjectileActor`, 또는 이를 상속한 유탄 BP |
| Projectile Mesh | 준비한 유탄 메시. 실제 에셋 미정 |
| Speed | 양수(cm/s). GrenadeLauncher가 곡선 위를 이동하는 실제 속력 |
| Arc Height | 0 이상(cm). 출발점·목표점 중 높은 위치 위로 솟는 정점 높이. 구체 값 사용자 지정 |
| Lifetime | 양수. 유탄의 실제 수명은 예상 도달시간+현재 프레임 시간보다 짧아지지 않도록 확장 |
| Collision Radius | 양수. 실제 유탄 크기에 맞춘 값 미정 |
| Gravity Scale | GrenadeLauncher 지정 곡선에는 적용하지 않음. 0이어도 발사 가능 |
| Use Ballistic Aim | 기존 값 로딩용으로만 유지·설정 화면 비노출. GrenadeLauncher 타입으로 포물선 자동 선택 |
| Aim At Target Ground | true. 이번 후속 C++ 반영 후 사용자 빌드 필요 |
| Favor High Arc | 구버전 값 로딩용 비노출 필드. 현재 유탄 높이는 ArcHeight로 지정 |
| Explosive | true |
| Explosion Radius | 양수. 최종 반경 미정 |
| Explosion Trace Channel | 기본 Visibility. 해당 채널을 Block하는 벽이 폭발 피해 차단 |
| Explosion VFX | 준비한 폭발 Niagara |

Lifetime·CollisionRadius·ExplosionRadius·ArcHeight의 신규 수치 임의 확정 없음. 반경 0·클래스 누락·유효 사거리 0·Speed 0·음수 ArcHeight는 설정 오류. ArcHeight 기본 0, 동일 높이 목표에서 양수를 지정해야 위로 솟는 곡선 생성. 수명 만료는 폭발 없이 회수, 충돌 시 기폭

### 사거리 기준 유탄 발사 후속 — 2026-10-24

- 사용자 확정: 실제 공격 거리 100m·무기 DA 사거리 조절·무기 종류에 따른 발사 형태. 기존 중력 기반 자동 속력이 너무 빠르다는 피드백 후 중력 없는 포물선·Speed 제어로 변경 요청, 곡선 높이는 유탄 DA에서 직접 지정 선택
- 최신 C++ 반영 후 설정: WeaponType=GrenadeLauncher, DeliveryType=PhysicalProjectile, EffectiveRange=10000. 유탄 DA Speed는 양수, ArcHeight는 원하는 정점 높이(cm). 별도 곡선 활성화 토글 없음. 지면 목표는 기존 AimAtTargetGround=true 유지
- EffectiveRange는 AI 공격 범위에만 사용. Speed는 곡선 위 실제 이동 속력, ArcHeight는 높은 쪽 끝점 위 정점 높이. World 중력·GravityScale·기존 FavorHighArc는 현재 곡선 계산·이동에 사용하지 않음
- 발사 순간 출발점·목표점·곡선을 고정, 이후 플레이어 추종 없음. 곡선을 충돌 반경 이내 오차의 구간으로 분할하고 모든 구간을 Sweep해 고속·저프레임에서 장애물 건너뛰기 방지. 접촉 시 기존 폭발 경로 사용
- 예상 비행시간=곡선 구간 길이 합/Speed, 실제 수명=max(DA Lifetime, 예상 비행시간+현재 프레임 시간). CurvePoints도 Commit→Factory→Actor에 전달, DA 공유 값 불변. CurvePoints 없는 일반 발사체는 기존 ProjectileMovement 사용
- 곡선 끝점에 충돌 없이 도착하면 폭발 없이 회수. Pool 회수 시 배열·진행 인덱스·속력·Actor Tick 정리. 목표 지형이 사라진 경우의 공중 폭발을 임의로 만들지 않음
- 정적·수치 확인: 높이·높이차·속력·프레임 간격 126개 조건에서 정점·이동 거리 대조. 분할 오차는 충돌 반경 5cm 예시에서 최대 약 4.84cm. 독립 계산 확인이며 UE 테스트 실행 아님. 최신 C++ 빌드·UHT·Editor·Automation·PIE 근거 없음
- 사용자 후속: Editor 종료 후 Astra_SoulLinkEditor / Win64 / Development 빌드. ArcHeight 설정 후 Speed를 바꾸며 근거리·50m·100m 및 높이차에서 비행시간·접촉 폭발 확인. GravityScale=0/1이 유탄 이동에 영향 없는지, 수명 만료·Pool 재사용·일반 NPC 동시 발사 회귀 확인
- 고정 ArcHeight가 높으면 근접 사격에도 상승·하강 경로가 생겨 최소 비행시간 증가. 근접 고위험 의도에 맞는 높이·속력은 사용자 튜닝 대상
- 충전·탄착 경고 UI는 아직 후속 범위. 예상 비행시간 데이터 추가를 실제 UI 표시 완료로 취급하지 않음

지면 조준은 현재 타겟 XY 아래의 첫 지면 좌표 사용. 자기 몸과 타겟 몸을 제외한 AttackTraceChannel Trace, 최대 깊이는 무기 EffectiveRange. 바닥은 해당 채널을 Block하도록 설정. 공중 타겟도 현재 XY 아래 지면 탐색, 예측 착지점 계산 없음. 지면 검출 실패 시 몸통 좌표로 대체하지 않음

최종 충돌 규칙: Pawn·지형에 닿으면 그 위치에서 즉시 폭발. Pawn 통과·튕김 없음. 플레이어가 유탄 경로에 들어가면 지면에 닿기 전에도 폭발 가능

### 보스 ABP·GA 연결

1. `/Game/Astra/Entity/Boss/ABP_Boss`의 부모 `AstraHumanoidAnimInstance` 유지
2. Class Defaults → Animation | Actions → Action Animations → Fire Montages: Key=`GrenadeLauncher`, Value=보스 Skeleton용 발사 Montage
3. Montage의 발사 프레임에 Montage Notify `Weapon_FireCommit` 설정. 기존 발사 Notify 계약 재사용, 별도 BP SpawnActor·탄약 소비 노드 추가 없음
4. Montage Slot과 AnimGraph Slot 일치 확인. 기존 Upper/FullBody 정책에 맞춘 슬롯 사용
5. 기존 NPC의 무기 발사 GA와 Upper Task 계약 재사용. 무소모 보스에 ReloadMontage·Reload GA 추가 불필요. 기존 공통 GA 클래스 자체의 변경 없음
6. BP·ABP·Montage·StateTree Compile 후 PIE

### 발사는 되지만 몽타주가 보이지 않는 경우 — 2026-10-24

> 최신 디버그 구현은 `Utils/Debug/UAstraDebugSubsystem`으로 이전. 아래 직접 콘솔 변수는 호환 유지하며 `astra.Debug`로 전체 설정/사용처를 확인한다. 출력 카테고리는 LogAstraDebug. [중앙 명령·검증 상태](../03_시스템%20설계/디버그%20서브시스템.md)

- 사용자 확인: 최신 유탄 발사 성공. 보스 몽타주 미표시·일반 NPC 몽타주 불안정, AttackCooldown=0.3. ShootType 후속 구분: 보스 테스트 SingleShot, 일반 NPC 라이플 RapidFire. 보스 ABP는 아직 작업 전
- 현재 SingleShot은 쿨다운 만료로 몽타주를 강제 정지하지 않음. FinishFireCooldown의 Montage_Stop(0)은 한 발 요청 RapidFire 전용. 0.3초 설정만으로 SingleShot 미표시 원인 확정 없음
- 정상 발사 경로는 Montage_Play 성공→Weapon_FireCommit Notify→실제 탄 생성. 발사가 된다는 것은 이 경로에서 몽타주 시간 재생과 화면 포즈 기여를 구분해야 한다는 근거. Slot·최종 출력 연결·상체 마스크·다른 포즈의 덮어쓰기 확인
- 보스 설정 대상: ABP_Boss의 ActionAnimations.FireMontages[GrenadeLauncher]에 보스 Skeleton 호환 Boss_Fire_Montage 연결. Content 경로에 두 에셋 존재만 확인했으며 실제 연결·호환성·Slot·블렌드 수치는 Editor 확인 대상
- Montage Track은 DefaultGroup.UpperBody, ABP의 Slot 노드는 UpperBody로 일치. 로코모션 출력을 Base Pose와 Slot 입력으로 보내고, Slot 결과를 Layered Blend per Bone의 상체 포즈로 합성. 필터 본은 보스 Skeleton의 실제 몸통 본 선택, 다른 Skeleton의 본 이름을 추정해서 지정하지 않음
- 일반 NPC는 기존 UpperBody Slot→AimOffset→상하체 합성→기존 FullBody Slot→Output 경로 유지. 중복 Slot 추가 없이 실제 몽타주와 이름·그래프 활성 경로 비교
- SingleShot 실제 간격에는 몽타주 종료·다음 Notify까지의 시간이 포함될 수 있음. AttackCooldown은 최소 발사 간격이며 0.3초가 몽타주 길이나 정확한 반복 주기를 뜻하지 않음
- 후속 스크린샷 MM_Rifle_Fire_Montage: 30fps·약 16프레임(약 0.53초), Weapon_FireCommit은 시작 0프레임 부근, Slot=DefaultGroup.UpperBody, Notify Tick Type=Queued, Auto Blend Out 활성화 확인. Blend In/Out 시간 항목은 접혀 있어 0.1초 수치는 화면에서 직접 확인 불가, 사용자 설명값으로 구분
- RapidFire C++은 FireEquippedWeapon에서 한 발 요청 후 입력 완료 처리. Commit 후 쿨다운 만료 시 FinishFireCooldown에서 Notify·종료 Delegate 해제→Montage_Stop(0.f)→FinishFire(true). 에셋 Blend Out을 0초로 덮으므로 0.3초 부근의 포즈 단절 요인 확인. 이 분기를 보스 SingleShot 미표시 원인으로 확대하지 않음
- 사용자 승인 후 C++ 반영: RapidFire 후반 정리에 max(0, Montage.BlendOut.GetBlendTime()) 사용, 기존 쿨다운·한 발 확정·Delegate 해제·Sequence 완료 시점 유지. 공통 RapidFire 경로라 Player/NPC 모두 영향. 정적 확인 완료, 이번 변경의 빌드·PIE 미수행. 그래프 Slot/가중치·실제 블렌드 값 확인과 실행 회귀는 별도 필요
- 플레이 확인: 라이플 Blend Out 실제 시간을 펼쳐 확인, 쿨다운 0.3에서 연속 발사 시 포즈가 0초로 끊기지 않는지 비교. 두 NPC 동시 발사·플레이어 RapidFire·탄창 소진/Reload·공격 취소 후 이전 콜백이 다음 발을 종료하지 않는지 확인. SingleShot 보스 미표시는 ABP 작업과 별도 유지
- 사용자 후속: Blend Out 보정 후에도 일반 NPC 몽타주 미해결. 탄은 나가지만 포즈가 갑자기 튀며, 쿨다운을 약 0.53초보다 길게 해도 동일. 0초 강제 종료만으로 해결된 문제로 기록하지 않음
- 사용자는 발사 종료→사격 대기 자세 복귀→다음 발사 흐름 검토. Slot은 원래 Source Pose로 복귀하므로 라이플 Idle ADS가 안정적으로 계속 평가되는지, 발사 Action 종료와 함께 비무장/로코모션 포즈로 전환되는지 우선 확인
- C++의 bWeaponUpperBodyActive는 무기 손 부착 여부, bEnableWeaponAimOffset은 손 부착+Carry Ready 기준. 일반 NPC ABP의 실제 Bool/Enum 연결은 아직 화면으로 확인하지 못함. IsAttacking만으로 상체 포즈를 켰다 끄는 연결과 구분
- 확인 후보: 라이플 Idle ADS→UpperBody Slot.Source→라이플 AimOffset.Base Pose→기존 상하체 합성→기존 최종 Slot/Output. Slot.Always Update Source Pose를 켜고 비교하여 몽타주 중 원본 포즈 평가가 중단되는 영향 구분. 정답으로 확정된 원인 아님
- Content에 MM_Rifle_Idle_ADS·MF_Rifle_Idle_ADS·Rifle_Idle가 모두 존재. 실제 NPC의 선택 클립과 MM_Rifle_Fire의 Skeleton·Additive Base Pose·첫/끝 포즈 호환성 확인 필요. 자산 이름 존재를 실제 연결 증거로 사용하지 않음
- 최신 로그의 Weapon_FireCommit 확정 실패 1건은 Owner 식별 없음. NPCCombat/WeaponShot 진단 로그가 없어 해당 NPC의 취소 원인으로 지정하지 않음. 이번 조사에서 C++ 추가 수정·빌드·Editor 실행 없음
- 사용자 추가 재현: Always Update Source Pose 비교 후에도 동일, 약 10~20발당 한 번 동작이 보이며 포즈가 튐. 해당 옵션을 해결책으로 지정하지 않음
- 사용자 제공 ABP_Humanoid_NPC AnimGraph 텍스트 확인: MM_Rifle_Idle_ADS → 무기 Enum → UpperBody Slot → WeaponSlotPose 캐시 → Rifle AimOffset → 무기 Enum → bWeaponUpperBodyActive Bool → Layered Blend Per Bone(가중치 1, spine_01/양팔) → 시각 보정 → FullBody Slot → Output 연결. Rifle AimOffset.BasePose가 WeaponSlotPose를 사용하며 공격 Action 기반 Bool 연결은 없음
- 실제 라이플 대기 SequencePlayer는 bLoopAnimation=False. 반복 대기 클립이라면 Loop 활성화 대상이나, 이 설정만으로 발사 몽타주가 드물게 보이는 원인 확정 없음. 붙여넣기 텍스트의 UpperBody Slot에는 bAlwaysUpdateSourcePose=True가 없으나 사용자의 비교 실패 보고는 별도로 유지
- 다음 분리 확인: 임시 Use Cached Pose(WeaponSlotPose)를 Output Pose.Result에 직접 연결하여 Slot 이후 AO/Bool/상체 합성/최종 Slot을 우회. 매 발사 동작이 보이면 후단 합성 경로, 그대로면 몽타주 실행·중단/클립/메시 갱신 경로로 범위 축소. 진단 후 원래 Output 연결 복구. 이 결과는 아직 미확인, 추가 C++·빌드·Editor 실행 없음
- 최신 사용자 정정: 관찰한 상체 꺾임은 공격→추격 전환에서 발생하며, 발사 몽타주가 화면에 재생되지 않는 문제도 별도로 유지. 추격 꺾임을 발사 몽타주 튐으로 혼동하지 않음. NPC는 추격 시 저장된 위치를 바라보며 이동해야 한다는 요구 확인
- 현재 C++는 추격 이동 요청에 LastKnownTargetLocation을 사용하지만, 가시 타겟이면 UpdateChaseMove/IssueChaseCommand에서 SetFocus(CurrentTarget)를 사용. 기억 이동에서는 SetFocalPoint(LastKnownTargetLocation)를 사용. 위치 기억·이동 목적지와 시선 설정이 서로 다른 경로임. 가시 중 기억 위치 갱신/상실 후 동결은 기존 정책 유지 대상
- 발사 경로는 StartFireMontage → Montage_Play → Weapon_FireCommit → CommitInitialShot → 실제 탄 생성. 탄 생성은 Notify 수신을 뒷받침하지만 최종 뼈 포즈 반영/몽타주 정상 끝까지 재생을 증명하지 않음. Actor/몽타주 인스턴스 ID/재생 위치·가중치/중단 사유를 함께 추적하기 전 조기 취소나 그래프 합성을 확정 원인으로 지정하지 않음
- 후속 사용자 승인: 추격 저장 위치 Focus와 몽타주 진단 C++ 반영, 이번 변경에 한해 빌드·에디터 실행·가능한 테스트 허용. 플레이어는 정상 표시되고 NPC에서만 발사 몽타주가 보이지 않음 사용자 확인
- 반영: UpdateChaseFocus로 추격 시선을 LastKnownTargetLocation에 통일, Controller/Character가 Actor 없는 좌표 Focus에서도 기존 보정 Yaw·회전 속도를 적용. 보이는 동안 기억 갱신, 상실 후 기억 동결, 공격 중 Actor Focus 정책은 유지
- 진단: `astra.Weapon.MontageDebug 1`은 Play/Notify/Cancel/End/CooldownStop 수명 로그, `2`는 0.1초 재생 샘플 추가. `WeaponMontage`는 Owner/플레이어 조종 여부/메시/ABP/몽타주 경로/InstanceId/NotifyId/재생 위치/가중치/상체 활성/Pitch/메시 Tick 조건 기록. `WeaponMontageSlot`은 실제 몽타주 Slot별 Node/Local/Global 가중치 기록. Slot 가중치는 직전 프레임 버퍼일 수 있어 시작 프레임 0만으로 원인 확정 금지. Notify ID는 진단용이며 수용 조건 변경 없음
- `astra.NPC.CombatDebug 1`은 NPCCombatStop의 취소 사유와 NPCChaseFocus를 포함. 추가 NPCFiringLineBlocked는 차단 액터/클래스/Owner/Instigator/컴포넌트/거리/초기 겹침을 기록. 동일 차단 액터 로그는 0.25초 간격, 게임플레이 사선 판정은 변경 없음
- Development Editor 빌드·UHT 성공: `Saved/Logs/Codex_NPCChaseMontage_Build.log`. 차단 액터 진단 보강 재빌드 성공: `Saved/Logs/Codex_NPCChaseMontage_BlockerBuild.log`. 기존 투사체 Automation 3개 Success/실패 0/경고 0, 보고서 `Saved/Automation/NPCChaseMontage/index.json`. 신규 테스트 파일 없음, 이 3개는 몽타주/추격 회귀 테스트가 아님
- 첫 실제 Editor PIE: `Saved/Logs/Codex_NPCChaseMontage_Editor.log`. 현재 맵의 BP_NPCCharacter_C_0/DA_BasePistol에서 11회 Play/Notify/CancelRequested, EndCompleted 0. 발사 직후 Position=0.0167에서 NPCCombatStop Reason=FiringLineBlocked와 0.05초 취소 확인. UpperActive=1, Slot NodeWeight=1이며 첫 샘플 Montage 가중치 0.2593. 이 재현은 Slot 누락보다 발사 직후 사선 취소 경로를 뒷받침. 라이플의 동일 원인과 차단 액터는 추가 확인 대상
- PIE에서 별도 기존 설정 오류: 플레이어 ABP에 ActionAnimations.DeathMontages 누락. 플레이어 사망 시 발생했으며 이번 발사 몽타주 문제와 구분, 해당 에셋은 변경하지 않음
- 차단 액터를 포함한 두 번째 실제 PIE: `Saved/Logs/Codex_NPCChaseMontage_BlockerEditor.log`. NPCFiringLineBlocked에서 BP_TestBulletActor_C_0/SphereComp, BlockerOwner·BlockerInstigator=BP_NPCCharacter_C_0, Distance=0, StartPenetrating=1 반복 확인. 방금 생성된 자신의 투사체가 NPC 사선 장애물로 잡혀 발사 몽타주가 약 0.017초에 취소되는 구체 원인 확인
- 추가 수정안: NPC 사선 LineTrace에서 자신 소유 AProjectileActor만 QueryParams.AddIgnoredActor 후 같은 구간을 재검사. 뒤의 벽/다른 액터는 여전히 장애물, 투사체 실제 충돌 프로필·피해·폭발은 변경하지 않음. 진단 승인 범위와 구분해 사용자 추가 승인 대기
- 사용자 추가 승인 후 위 자기 투사체 제외를 NPCCharacter::HasClearFiringLine에 반영. 취소 조건을 무조건 완화하지 않고 사선 질의에서 자신의 탄만 건너뜀. 재빌드 첫 시도는 실행 중 Editor의 DLL 잠금으로 LNK1104, Editor 종료 확인 후 재시도 진행
- 최종 사선 수정 빌드 성공: `Saved/Logs/Codex_NPCChaseMontage_FixBuild_Retry.log`. 최신 기존 투사체 Automation 3개 Success/실패·경고 0: `Saved/Automation/NPCChaseMontageFinal/index.json`. 에디터 재실행 후 Codex 화면 자동화는 사용자 Escape로 중단, 이후 사용자 직접 플레이에서 몽타주 재생 확인. NPC 발사 몽타주 미표시 해결 지정
- 최신 잔여는 추격 시 상체 블렌드가 꺼지는 듯한 자세 변화. 그래프는 손 부착 기반 bWeaponUpperBodyActive를 사용하고 Layered Blend 가중치 1. 최신 로그의 발사 수명 샘플에서 UpperActive=0은 없지만 추격 전체의 Bool 유지 증거로 확대하지 않음
- 원인 후보의 코드 근거: AAIController::UpdateControlRotation은 FocusActor가 Pawn이 아니면 Control Pitch를 0으로 만듦. 저장 좌표 SetFocalPoint는 Actor가 없으므로 이 경로 적용. NPC의 기존 시야 상실 중립화도 Pitch를 0으로 만듦. 공격↔추격에서 조준 상하각이 중립으로 바뀌는 경로가 존재하며 ABP 상체 Bool 비활성화와 구분
- 기존 승인된 저장 위치 바라보기 보정: 유효한 저장 좌표로 실제 추격 중(가시 타겟 또는 LastKnownMove)이면 Super 갱신 후 `(LastKnownTargetLocation - NPC->GetPawnViewLocation()).Rotation().Pitch`를 Control Rotation에 복원. 몸은 기존 FaceRotation의 Yaw만 회전, AimOffset은 기존 AimPitchLimit·보간 사용, 상체 Bool 변경 없음. LostDelay/Searching 등 실제 추격 외 중립화는 유지. 백업 `Saved/CodexChanges/NPCChasePitch-20261024-191849`, 이번 Pitch 보정의 빌드·UHT·Editor·PIE 미수행
- 사용자 추가 특정: 높은 위치의 일반 NPC가 공격 중 타겟을 놓쳐 추적으로 전환하는 순간 AO 중심 복귀 또는 Idle/기본 자세처럼 보임. 실제 선택 경로는 사용자도 미확인, 여전히 미해결. 이전 보정 완료/사용자 재생 확인은 발사 몽타주에만 해당
- 이전 Pitch 보정의 누락 확인: LoseTrackedTarget이 ClearCombatAim으로 Control Pitch를 즉시 0으로 만들고 LostDelay도 복원 조건에서 제외. LastKnownMove에 진입한 뒤만 고치는 것으로 전환 순간 중립화를 막지 못함
- 후속 C++ 보정: LoseTrackedTarget은 발사용 Weapon Aim만 무효화하고 즉시 저장 좌표 Focus 적용. UpdateControlRotation은 LostDelay/LastKnownMove의 기억 위치를 이동 Task 활성화 사이 공백에도 유지하고 Pitch 복원. 실제 보이지 않는 타겟 좌표를 새로 읽거나 발사를 허용하지 않음. 이동 대기시간·수색/포기·피격 바라보기·상체 Bool·자동 수납 정책은 유지
- 새 선택 진단 `astra.NPC.PoseDebug 1`: NPCPose를 0.1초 간격으로 기록. Brain/Phase/Visible/ChaseMove/StoredFocus/HasMemory/Focus/Saved/Carry/InHand/UpperActive/AimEnabled/WeaponType/ControlPitch/AimPitch/Speed 포함. 기본값 0, 진단 후 `astra.NPC.PoseDebug 0`. ABP 값은 마지막 AnimInstance 갱신 스냅샷이므로 전환 직후 한 줄보다 전후 연속 샘플로 판정
- 판독: UpperActive=0이면 제공 그래프의 기본 로코모션 분기 선택. UpperActive=1/AimEnabled=0이면 무기 상체 포즈는 유지하되 AO 목표 Pitch가 0. UpperActive=1/AimEnabled=1에서 AimPitch가 0으로 돌아가면 AO 입력 경로 확인. Carry/InHand로 자동 수납·손 부착 변화도 함께 구분. 자동 수납이 실제 원인인지는 아직 확인하지 않았으므로 해당 정책을 임의 변경하지 않음
- 백업 `Saved/CodexChanges/NPCLostTargetPose-20261024-192958`. 선언·정의·호출 및 차이 공백 정적 확인. 이번 LostDelay 보정·PoseDebug의 빌드·UHT·Editor·PIE 미수행, 높은 NPC의 상실→LostDelay→LastKnownMove와 재발견 전환 검증 전
- 최신 사용자 확인: 추적 상체 문제 해결. 높은 위치의 NPC 타겟 상실→추적 포즈 문제 해결 지정. 위 빌드/실행 미수행은 당시 Codex 작업 근거이며 사용자 확인 이전의 상태다. 다음 작업은 보스 ABP 구성으로 이동, 이 문제 진단을 다시 우선 작업으로 제안하지 않음

- 보스 최종 ShootType은 ChargedShot으로 재확인. 현재 테스트 SingleShot과 구분. ChargedShot은 enum·설정값만 선언되어 있고 충전 실행·머즐 구체 성장·탄착 경고는 구현 전

```text
보스 로코모션 포즈 ─────────────────────→ Layered Blend per Bone.Base Pose
보스 로코모션 포즈 → Slot(UpperBody) ──→ Layered Blend per Bone.Blend Pose 0
상체 합성 결과 → 기존 최종 포즈 경로 → Output Pose

공격 요청 → Fire Montage 재생
  ├─ 정상: Slot 포즈 반영 → Weapon_FireCommit → 탄 생성 → 정상 종료 → Ready
  ├─ 실패: 에셋/재생 전제 오류 → 탄 생성 없음 → 기존 GA/Task 실패 정리
  └─ 취소: GA/Task·몽타주 종료 정리 → 확정 전 추가 탄 생성 없음
Slot 경로 미연결: Notify로 탄은 생성되어도 최종 화면 포즈에는 발사 애니메이션 미반영 가능
```

```text
Entity 초기화 → 보스 WeaponInstance 생성
→ NonAmmoProjectileData·메시·소켓·ABP 부착 설정 검사
   ├─ 실패 → Error → 장착 실패 → 필수 설정 수정
   └─ 성공 → 손 부착·Ready
      → Brain 공격 요청 → Upper Task → GA
      → 현재 타겟 아래 지면 탐색·정확한 목표점 갱신
         ├─ 타겟 상실·지면 없음 → 조준 무효·발사 시작 실패
         └─ 지면 검출 → 기존 Montage 조준 스냅샷에 좌표 저장
      → FireMontages[GrenadeLauncher] 재생
         ├─ 누락·재생 실패 → GA/Task 실패·Ready 복귀
         └─ Weapon_FireCommit
            → 정의·실행 조건 재검사 → 고정 곡선 구성
               ├─ 정의·곡선 오류 → Commit 실패·탄약 소비 없음
               └─ 성공 → Factory·Pool 발사 → 기존 쿨다운
                  ├─ 충돌 → 이동·충돌 중단 → 범위·벽 검사
                  │          → Actor당 피해 1회·폭발 효과 → Pool 회수
                  └─ 수명 만료 → 피해·폭발 없이 Pool 회수
발사 확정 전 취소·사망 → 기존 GA/Task·몽타주 정리, 추가 생성 없음
발사 확정 후 취소 → 이미 생성한 유탄은 충돌·수명에 따라 종료
월드 종료 → Timer 정리
```

### 확인 항목

- 탄약 등록·탄창·Reload 없이 1초 쿨다운 기준 반복 발사
- 일반 NPC의 기존 탄창 소비·정상 Reload 유지
- 포물선과 실제 착탄 위치, 수명 내 착탄 여부 확인. 이동 타겟 선행 조준은 이번 범위 밖
- 범위 안/밖·벽 뒤·직격 대상·복수 충돌 컴포넌트의 피해 1회 확인
- 소유자 피해 제외, 무적·방어력·사망 경로 유지
- 재사용 유탄의 이동 컴포넌트·소유자·이전 Timer 상태 잔존 없음
- `astra.NPC.CombatDebug 1` 로그는 같은 Owner 기준 확인. 진단 후 0으로 해제

근접형 좀비·아스트라의 실제 공격 판정, 착탄 경고 UI, 회차 정리·특수 패턴은 후속 범위

### C++ 검증 근거

- 2026-10-24 `Astra_SoulLinkEditor / Win64 / Development` 전체 빌드·UHT 성공. 로그: `Astra_Unreal/Saved/Logs/Codex_BossProjectile_Build.log`
- 헤드리스 `Astra.Combat.Projectile` Automation 3개 성공: 무소모/등록 탄약 소유권, 발사체 정의 유효성, 실제 포물선 발사·범위 피해 중복 방지·벽 차단·Pool 재사용·직접 정리
- 보고서: `Astra_Unreal/Saved/Automation/BossProjectile/index.json`, 2026.10.24-00.53.16 UTC. 실패 0, 경고 포함 성공 1: 기존 GameplayCueNotifyPaths 미설정 경고
- 테스트용 임시 World와 데이터 사용. 실제 보스 DA·ABP·StateTree 에셋의 Compile·PIE 통과와 구분
- 위 빌드·Automation 결과는 무소모 발사·폭발 구현분. 이후 추가한 지면 조준 코드는 포함하지 않음. 후속분은 선언·정의·호출 경로·차이 대조만 수행, 사용자 빌드·Compile·PIE 대기

> 갱신: 2026-10-21. 기존 공격 반복·기억 추적·순찰/피격·발사 정렬/사선·착지 복구 안내 5건 통합
> 사용자 확인: 추적·재장전 완료. 사망 전이 해결·우선권 상향. 애니메이션 수정 후 PT-00-3 패스 지정
> 아래는 현재 연결 기준과 회귀 확인표. 과거 진단을 현재 미연결·재작업 요구로 해석하지 않음

## NPC 명중률 연결 검토 — 2026-10-24

- 사용자 확정: 원뿔 산포 방식 유지, NPC 종류·성향으로 명중률 보정, 적용 대상은 NPC 직선 총기. 플레이어·보스 유탄·Beam의 실제 탄도는 이번 연결 범위에서 제외
- 명중률은 무기 내구도와 관계있다는 기존 의도 재확인. 내구도 감쇠는 무기 DA 커브로 지정하되 현재는 구현하지 않도록 사용자 명시. 내구도 비율을 곧바로 명중률에 선형 곱하는 정책으로 임의 대체하지 않음
- 현재 코드: 공통 RecoilHandler는 NPC 반동 수치도 누적하지만 NPC 조준 회전에는 미적용. 사용자 승인 후 명중률을 NPC 직선 총기의 실제 발사 방향에 연결. EquipmentInstance의 기존 내구도·파손 검사는 유지, 명중률 내구도 커브는 없음
- C++ 반영: 무기 BaseAccuracy×AIAccuracyMultiplier×Entity DA.NPCTypeAccuracyMultiplier×CombatTendencyData.AccuracyMultiplier에서 자신/대상 이동 페널티를 차감하고 0~100 제한. 배율 초기 1은 중립값이며 종류·성향별 구체 수치는 사용자 지정
- 원뿔 반각은 기존 MaxSpreadAngleDegrees×(1-최종 정확도/100). 발사 확정에서 1회 방향 샘플링, 개별 탄은 정해진 직선으로 이동. 물리 투사체 초기 속도·SpawnRotation·AttackSpec 방향과 Hitscan 레이에 같은 산포 방향 적용. 명중/빗나감 추첨·피해 강제 적용 없음
- AIMissSpreadAngle은 사용하지 않음. 원뿔 상한은 기존 MaxSpreadAngleDegrees이며 별도 빗나감 확률 모델 없음. 조준 유지 시간에 따른 정확도 상승·NPC 카메라 반동 추가 없음. 내구도 커브는 후속 범위
- AIController가 자신의 NPC Entity DA·성향과 가시 타겟 이동을 조회해 계산, WeaponComponent Facade가 반각을 요청에 전달. AttackCommitHandler는 NPCController/성향을 직접 조회하지 않고 반각으로 방향을 한 번 샘플링. Hitscan은 기존 목표 거리의 레이를 같은 방향으로 회전, 실제 충돌만 피해 적용
- AIController 조종 NPC의 Ranged·PhysicalProjectile/Hitscan에만 적용. 플레이어 조종·GrenadeLauncher 지정 곡선·Homing·Beam·근접은 제외. 조준 기준점과 NPC 몸 방향 판정은 변경하지 않음
- 정적 확인·명중률 수치 대조 완료. 신규 테스트 파일·UE 테스트 실행 없음. 이번 명중률 C++ 빌드·UHT·Editor·PIE 미수행

사용자 DA 설정:

| 위치 | 필드 | 초기값/역할 |
|---|---|---|
| NPC Entity DA → NPC / Accuracy | NPC Type Accuracy Multiplier | 1=종류 보정 없음 |
| NPC Entity DA → Combat Tendency Data | Accuracy Multiplier | 1=성향 보정 없음 |
| 무기 DA → Weapon / Accuracy | Base Accuracy, Max Spread Angle Degrees | 기본 정확도와 최대 원뿔 반각 |
| 무기 DA → Weapon / AI Accuracy | AI Accuracy Multiplier | NPC용 무기 보정 |
| 같은 AI Accuracy | AI Moving / AI Target Moving Accuracy Penalty | 자신·대상 이동 시 각각 차감 |

종류·성향 수치는 사용자 지정, 에셋 자동 수정·종류명별 하드코딩·랜덤 성향 부여 없음. 성향은 Possess 시 복사하므로 설정 변경 후 PIE 재시작으로 재초기화

검증 예시: BaseAccuracy=80·배율 모두 1·최대 반각 10°이면 정지 80/2°, 자신 이동 페널티 20이면 60/4°, 대상 이동 페널티 15이면 65/3.5°, 모두 이동이면 45/5.5°. 정확도 상한 100은 산포 0°, 하한 0은 최대 반각이며 반드시 빗나감을 뜻하지 않음

Editor 종료 후 Development Editor 빌드·재시작, 두 NPC의 종류·성향 배율을 다르게 설정해 정지/이동/대상 이동 사격 비교. PhysicalProjectile/Hitscan 방향·실제 충돌, Player·유탄·Beam 제외, Reload·취소·동시 발사 회귀 확인. `astra.NPC.CombatDebug 1`에서 성공한 발의 `NPCShotAccuracy Owner=... Accuracy=... SpreadHalfAngle=...` 로그 확인

## 문서 역할

- 행동 정책·C++ 책임: [NPC 행동](../03_시스템%20설계/NPC%20행동.md)
- 공통 입력·상하체·GA 계약: [StateTree와 GAS](../03_시스템%20설계/StateTree와%20GAS.md)
- Death Task·GA·전신 Slot·제거: [공통 사망 연결](공통%20사망%20GA·몽타주·제거%20연결.md)
- 현재 착수 범위·별도 잔여: [현재 작업 상태](../01_일정%20관리/현재%20작업%20상태.md)

빌드·프로젝트 파일 재생성·Editor 편집은 사용자 수행. 이 문서 정리의 빌드·UHT·Editor Compile·PIE 수행 없음

## 공통 바인딩과 트리 배치

| 위치 | Task·설정 | 바인딩·완료 기준 |
|---|---|---|
| NPCBrain Root | NPC Update Target Tracking | Context.AIController. 완료 판정 제외, 지속 Running. 별도 Controller Tick·BP Timer 중복 추가 없음 |
| NPCBrain Idle | 기존 BrainState/추적·공격 종료 Task + NPC Idle Command | 동일 AIController. Idle Command는 완료 판정 참여·Running. Base Idle에 배치하지 않음 |
| NPCBrain Attack | NPC Set Brain State(Attack), NPC Set Chase Command(false), NPC Combat Attack | 동일 AIController. 지속 Combat Task만 완료 판정 참여, TasksCompletion=All. 실패 복구→Idle |
| Base Ground 부모 | 실제 에셋 STT_NPC_GroundBase, 초기 안내명 STT_NPC_GroundReady | 공통 GroundedEntered 알림. 아래 첫 Tick·완료 설정 적용 |
| Base Patrol / ChaseWalk / HitReaction | 각 이동·피격 방향 Blueprint Task | 해당 NPC의 부모 Pawn·EntityContext 재사용. 외부 Player 조회 없음 |
| Upper Attack / Ready / Reload | EntityContext 기반 공통 실행 계약 | 폴더명만으로 Task 분리 강제 없음. ReadyEntered와 GA 완료 수명 유지 |

필수 Controller·Context·EventBus 누락은 바인딩·초기화 오류로 진단. 정상 작업처럼 조용히 건너뛰지 않음. 종료 중 이미 파괴된 Controller 접근과 실행 중 필수 참조 누락은 구분

## 다중 NPC 무기 Handler 소유 — 2026-10-24

- 사용자 재현: 일반 NPC 두 개 중 한쪽만 발사. Brain Attack 상태와 실제 Fire/Commit 구분
- 기존 로그 단서: 서로 다른 몸 방향의 일반 NPC에서 동일한 총구 방향 관찰. 공유 참조 의심 단계이며 수정 전 Handler 주소·Outer 증거는 없음
- 공격 요청 C++은 GetPawn으로 자신의 Pawn에 전달. WeaponComponent Handler 6개에 Instanced 누락 확인
- 사용자 승인 범위: Handler 6개의 Transient, Instanced 선언, 초기화 시 Outer·템플릿·Context Owner 검사, 최초 소유 진단 로그. 사거리·StateTree·에셋·유탄 탄도 정책 변경 없음
- 빌드·Editor 실행은 이번 변경에 한해 사용자 명시 요청. 초기 UBA 기본 작업 경로 접근 실패 후 프로젝트 Saved/UBA/HandlerOwnership 경로 지정으로 재시도, Astra_SoulLinkEditor / Win64 / Development 빌드 성공. UHT 생성 프로퍼티의 Instanced 플래그 반영 확인
- 테스트 맵 헤드리스 실행: 일반 NPC C_2·C_4, 보스 C_5, 플레이어 C_0 각 Handler 6개. Handler 주소 24개 모두 고유, Context 주소 4개, Outer·Context Owner 불일치 0. 소유 검사 실패·Assertion 없음
- 실행 근거: Unreal Saved/Logs/Codex_HandlerOwnership_Runtime.log, Saved/CodexChanges/HandlerOwnership-20261024-105153/RuntimeSummary.json. 빌드 근거: Saved/Logs/Codex_HandlerOwnership_Build_Retry.log
- 일반 Editor 테스트 맵 로드 완료. 에셋 편집·저장·Automation·PIE 미수행. 헤드리스 소유 검증을 두 NPC의 동시 발사 해결 증거로 확대하지 않음
- 사용자 후속 확인: 두 NPC 동시 발사 성공. 다중 NPC 중 한쪽만 발사하는 문제 해결 지정
- 원인 판단: Instanced 누락으로 무기 Handler의 컴포넌트별 인스턴스 소유가 유지되지 않는 문제. 분리 선언 후 고유 Handler·Context 실행 확인과 동시 발사 사용자 확인이 근거. 수정 전 공유된 객체 주소나 덮어쓰기 순서까지 직접 확인한 것은 아님
- 보스 No ballistic solution은 실행 중 계속 관찰된 별도 잔여. 동시 발사 확인을 보스 탄도·충전·탄착 경고 검증 완료로 확대하지 않음

## 피격 인지·거리 진입·추격 경로 갱신 분리 — 2026-10-25

사용자 추가 재현: 플레이어가 추격 범위 안에서 움직이면 보스 걷기가 중간에 끊김. 코드에서 `UpdateChaseMove`가 이동 목표 갱신 때 `StopMovement()` 후 `MoveTo()`를 호출하던 경로 확인. ABP GroundSpeed/bShouldMove는 실제 속도 기준이므로 이 강제 정지가 걷기 포즈에도 영향을 줄 수 있다. 실제 PIE에서 원인이 하나뿐인지와 수정 효과는 아직 검증하지 않았다.

- 경로 갱신: 새 경로 계산 전 기존 이동을 정지하지 않음. 계산 성공 후 기존 요청 ID를 무효화하고 `MoveTo`로 교체하여 이전 요청의 동기 Aborted 콜백을 실패로 집계하지 않음. UE 5.8 `UPathFollowingComponent::RequestMove`의 새 요청 교체는 속도를 유지하는 경로를 사용한다.
- 중복 RequestChaseMove: 활성 추격이면 Update만 수행. 속도·요청·재시도 상태 초기화 생략.
- 실제 종료/실패: CancelChaseMove와 경로 실패는 현재 자신 소유의 이동 요청만 정지. 경로 재시도 한도·부분 경로 접근·수색/기억 만료 유지.
- 거리: RangeOverride=0에서 Controller의 가시 관측 갱신이 공격 거리 진입을 기억한다. 100m·비율 0.7이면 **70m 이하 관측 → 공격 거리 유지 상태 → 100m 초과에서 해제**. Brain이 아직 Chase이거나 공격 준비 중이어도 70~100m에서 거리 기준이 다시 70m로 돌아가지 않는다. 시야 상실·새 타겟·기억 포기는 초기화. 양수 RangeOverride는 기존 Brain 상태 기준 호환 경로이므로 이번 무기 기준 연결은 모두 0으로 유지.
- Native의 SetBrainState/SetChaseCommand/CombatAttack/IdleCommand/RootTracking Task는 `bShouldStateChangeOnReselect=false`. 공통 활성 상태의 재선택만으로 명령·공격 구독·순찰을 재시작하지 않는다. RootTracking Exit도 Sustained에서는 충전을 강제 취소하지 않음. 실제 상태 이탈·트리 종료·사망 취소는 유지.
- 피격: 생존 피격 사실은 항상 전달, 보스 false는 취소/HitReaction/DamageLook 생략. 미인지 보스는 첫 유효 공격자 위치만 기억해 Chase로 대응, 감지되지 않은 공격자 실제 위치를 계속 읽거나 바로 발사하지 않음. 일반 NPC true는 기존 반응을 실행.

### 사용자 Editor 연결

1. 최신 C++ 빌드 후 보스/일반 NPC Brain 트리를 Compile. Root의 지속 Task `NPC Update Target Tracking` → AIController를 해당 트리 Context Controller에 바인딩. 자식 Chase/Attack 전환 때 Root는 유지. Controller를 첫 NPC 검색 결과나 공용 변수에 바인딩하지 않음.
2. Brain Idle/Chase→Attack: 기존 가시 타겟·사거리·사선 조건 유지, `RangeOverride=0`, Invert=false. Attack→Chase 거리 전이는 동일 Condition의 Invert=true. 시야/사선 상실 전이는 별도 유지. AttackEnterRangeRatio는 보스 DA 0.7.
3. 해당 Chase→Attack 전이의 **Delay Transition 끄기**, DelayDuration/DelayRandomVariance=0. C++에는 추격→공격용 고정 지연이 없다. 무기 Draw/몸 정렬/UpperReady/발사 쿨다운 대기는 Attack 내부 실행 대기이며 전이 Delay와 다르다. 현재 uasset의 실제 Delay 값은 확인·수정하지 않았음.
4. 실제 Base ChaseWalk의 Blueprint 이동 Task Class Defaults에서 **Should State Change on Reselect 끄기**. 일반 경로 갱신마다 FinishTask/CancelChaseMove를 호출하지 않음. 도착·실패 상태값만으로 즉시 완료/Chase 자기 전이를 만들지 않고 Brain의 전이에 맡김. Brain의 시작 명령/SetBrainState 단발 Task는 Considered for Completion=false, 지속 Task만 완료 판정에 참여. OnStateCompleted→Chase/Root 또는 강제 Reactivate 설정이 있다면 의도한 재시작인지 확인.
5. 기존 우선순위 복구 유지: Death=Critical. 일반 피격 반응을 사용하는 Base는 DamageLook이 Chase 이동보다 우선. 보스에서 GroundState→ChaseWalk 우선순위 조정 후 추격 이동이 된 것은 사용자 확인 이력이며, 이번 목표 갱신 끊김과 별개.
6. Boss Entity DA `Can Interrupt Actions On Damage=false`. Blueprint OnDamageTaken에서 별도로 CancelAction/CancelChaseMove/StopMovement/충전 취소를 직접 발신하는 연결은 공통 Mediator 정책을 우회하므로 제거·정리 대상. 해당 BP 수정은 사용자 작업.

```text
Brain Root: NPC Update Target Tracking (Running)
  → 가시 타겟 관측: 70m 이하에서 거리 진입 기억
  → Chase→Attack 조건 통과: 100m까지 거리 유지 (시야/사선 조건 별도)
  → Attack: 무기 Ready·몸 정렬·쿨다운 대기 → 발사
  → 100m 초과/시야·사선 상실: 기존 Chase 전이

Base ChaseWalk Enter → RequestChaseMove
  Tick → UpdateChaseMove → 타겟 이동 감지
       → 새 경로 계산 성공 → 기존 속도 유지하며 요청 교체
       → 실패 → 소유 이동 정지 → 제한 재시도/기억 위치 수색
  실제 Exit → CancelChaseMove (해당 추격 요청만 정지)
  같은 활성 상태 재선택 → Enter/Exit 반복하지 않음

생존 피격 → 방향 사실 저장
  보스 false → 최초 미인지 공격자만 기억, 기존 이동/충전/발사 유지
  일반 NPC true → Upper 취소 + HitReaction → BeginDamageLook에서 이동/사격 정지
치명적 피해/트리 종료 → 기존 강제 취소·사망 처리
```

### 실행 확인 순서

`astra.Debug NPC.Combat 1`, 필요 시 `astra.Debug NPC.Pose 1`. NPCDecision의 Owner·Event·Brain·CombatActive·RangeEngaged·KnownDistance(cm)·ChaseStatus·Speed를 같은 보스 기준으로 비교.

- 70m 진입 뒤 71~99m 이동: RangeEngaged=1 유지. 최대 범위 안 거리 변화만으로 Chase/Attack 명령이 반복되지 않아야 함.
- 100m 초과 뒤 99m 복귀: Chase 유지, 다시 70m 이하에서 진입.
- 공격 범위 밖의 추격 중 플레이어 좌우/전후 이동: ChasePathRequested→ChasePathAccepted 전후 속도가 목표 갱신 때문에 0으로 떨어지지 않아야 함. 취소/실패 이벤트나 BrainStateEntered/ChaseCommandStarted 반복 여부로 경로 갱신과 트리 재진입 구분.
- 가림·경로 실패·끝점 도착·재감지·무기 변경·연속 피격·사망·PIE 종료 회귀. 보스는 피격 중에도 유지, 일반 NPC는 반응 유지.
- 정적/수치 검토 35개 확인은 C++ 실행 또는 PIE 통과 증거가 아님. 빌드·UHT·Editor·PIE 미수행, 신규 Source 테스트/uasset 편집 없음. 백업과 보고서: `Saved/CodexChanges/NPCDecisionSeparation-20261025-153036`.

## 두뇌 전이

### 일반 라이플 NPC의 감지 거리와 무기 사거리 — 2026-10-24

사용자 제공 실제 값:

| 설정 | 값 | 거리 |
|---|---:|---:|
| 무기 EffectiveRange | 10000 | 100m |
| Entity DA CombatTendencyData.SightRadius | 2000 | 20m |
| Entity DA CombatTendencyData.LoseSightRadius | 2500 | 25m |

- 현재 최초 타겟 획득은 20m 시야 범위에서 시작. 무기 사거리 100m 안에 들어오는 것만으로 타겟 획득 없음
- 감지한 타겟도 25m 이탈 시 가시성 상실 가능. 기억 유지와 실제 사격 허용은 별개
- 사거리 Condition과 실제 공격은 HasVisibleTarget도 검사. 가까워진 뒤 최초 발사가 시작되는 설정 차이 확인
- 사용자 후속 계획: 시야 10배 확대, SightRadius=20000(200m) 설정 예정. 기존 감지/이탈 간격을 유지하는 LoseSightRadius=25000(250m) 동반 확대 제안. 실제 Editor 적용·완료 확인 전이며 LoseSightRadius 최종 지정은 미확인
- 설정 변경 후 시야각·장애물·몸 정렬·무기 Ready 조건을 포함한 실제 발사 확인 필요. 현재 값 확인을 수정·PIE 완료로 기록하지 않음
- 보스 유탄의 고정 속도·중력·포물선 해 실패와 별개 원인
- 사용자 후속: 시야 확대만으로 사거리 내 미발사 해소 없음. 감지 거리 차이를 단독 원인으로 확정하지 않음
- 다음 확인: Brain의 Idle→Attack·Chase→Attack·Attack→Chase 사거리 Condition의 RangeOverride. 0 이하면 무기 EffectiveRange, 양수면 해당 값으로 전이 거리 덮어쓰기. 실제 공격 실행부는 무기 EffectiveRange 재검사
- ChaseAttackRangeFraction과 MoveAcceptanceRadius는 추적 이동의 도착 허용 거리. 공격 상태 전이의 RangeOverride와 구분. 이미 Brain Attack 활성인 경우에는 일반 NPC Owner의 NPCCombat 대기 사유로 실행 차단 지점 확인
- 사용자 추가 확인: 전이 사거리 RangeOverride 모두 0. 전이 거리 덮어쓰기 후보 제외. 시야 확대만으로 미발사 해소 없음, 무기 사거리·감지 거리 수치만으로 원인 확정하지 않음
- 후속 CombatDebug 재현에서 일반 NPC AttackRequested·AwaitAttackResponse 반복과 CanAttack/Facing/ClearFiringLine/UpperReady=1 확인. 이후 한쪽 NPC만 WeaponShot 기록, 다중 인스턴스 소유 진단으로 전환. 위 Handler 소유 항목 참조

모든 Condition의 AIController는 동일한 두뇌 Context에 바인딩. `NPC Has Target Memory`는 현재 Actor 참조와 별개이며 Actor 소멸 후 기억 위치 이동을 허용

| 출발 | 조건 | 목적지 |
|---|---|---|
| Idle | 가시 타겟·사거리 안 AND Clear Firing Line(Invert=false) | Attack. 포괄 Chase보다 먼저 평가 |
| Idle | Target Memory(Invert=false) | Chase. 중복 사선 차단 전이 불필요 |
| Chase | 가시 타겟·사거리 안 AND Clear Firing Line(Invert=false) | Attack |
| Chase / Attack | Target Memory(Invert=true) | Idle |
| Attack | Target Memory(false) AND 사거리 Condition(true) | Chase |
| Attack | Visible Target(false) AND Clear Firing Line(true) | Chase. 사거리 이탈 전이와 별개 |
| Root | 사망 명령 | Dead. Critical 우선권 유지 |

괄호의 true/false는 **Invert 값**. 사거리 Condition은 가시성도 검사하므로 벽 가림 시 일반 결과 false. 2026-10-25부터 Range Override=0이면 활성 무기 EffectiveRange를 최대 거리로 사용하며 현재 BrainState에 따라 성향 진입 비율 적용

### 공격 거리 경계 완충 연결 — 2026-10-25

1. 사용자 빌드 후 보스/NPC Entity DA의 Combat Tendency Data → `Attack Enter Range Ratio` 확인. 새 기본값 0.85. 0.85는 최대 사거리의 85%이며, 개별 NPC 성향에서 조절. 기존 일반 NPC에도 기본값 적용, Player 무기 사거리에는 적용하지 않음
2. 무기 DA EffectiveRange 유지. 10000cm이면 진입 8500cm, 이탈은 10000cm 초과. 비율 1이면 완충 없음. 성향은 초기화에서 복사하므로 DA 변경 후 PIE/NPC 재초기화 필요
3. 기존 `타겟이 사정거리 안에 있는가` Condition 유지. Idle/Chase→Attack의 Invert=false, Attack→Chase의 Invert=true. RangeOverride는 모두 0 유지. 양수는 최대 전이 거리 기준을 별도로 바꾸므로 실행부의 무기 기준과 다를 수 있음
4. 각 Brain 상태의 기존 NPC Set Brain State 값(Idle/Chase/Attack)을 맞게 유지. 최초 구현은 BrainState를 기준으로 판정했으나 최신 무기 기준(Override=0)은 관측된 거리 진입 기억을 사용. BrainState Task는 의도·추격/충전 정책에 여전히 필요하므로 생략하지 않음
5. 공격 진입의 시야·사선 조건, 타겟 기억·사망 전이 유지. 보스 충전은 추격 전환 후 기존 실행을 이어가며, 새 충전 시작은 이 거리 정책 사용. 후속 구현은 위 분리 절 기준: 무기 기준(Override=0)의 거리 진입 기억은 BrainState 진입 전에도 유지

| 현재 상태 | 거리 조건 | 결과 |
|---|---|---|
| Idle/Chase | 85m 이하 | 기존 시야·사선 조건과 함께 Attack 진입 가능 |
| Idle/Chase | 85m 초과 | 접근 유지, 99m로 돌아와도 즉시 Attack 재진입 없음 |
| Attack | 100m 이하 | 기존 공격 유지, 90m로 멀어져도 Chase 전환 없음 |
| Attack | 100m 초과 | 새 공격 요청 중단·일반 공격 취소, StateTree에서 Chase 전환 |

위 표는 EffectiveRange=100m·비율=0.85 예시. 이미 진행 중인 충전의 추격 유지·마지막 관측 위치 발사 예외는 앞선 규칙 유지

```text
Chase → 85m 도달 → Attack → 90~100m 왕복: Attack 유지
Attack → 100m 초과 → Chase → 99~101m 왕복: Chase 유지 → 85m 도달 시 Attack
실패/시야 상실 → 기존 사선·타겟 기억 전이, 충전 예외만 보존
사망/명시적 취소/트리 종료 → 기존 정리 경로 유지
```

거리 이탈 때 Controller가 Combat Task 구독을 먼저 해제하지 않도록 수정. 전이 전에 다시 범위 안으로 들어오면 같은 Task로 재평가하고, 실제 Exit에서 정리. 별도 전이 Timer·이동 사격 정책 추가 없음

진단은 기존 `astra.Debug NPC.Combat 1` 사용. 중앙 NPCCombat 로그에 Brain/KnownDistance/MaxRange/DecisionRange/EnterRatio 추가. KnownDistance는 저장 관측 좌표까지의 거리, 단위 cm. 가려진 타겟의 실제 위치는 로그에서도 새로 조회하지 않음

정적 검토·선언/정의/호출 대조 범위. 빌드·UHT·Editor·PIE·Automation 미수행, 신규 Source 테스트 파일·uasset 변경 없음. 백업 Saved/CodexChanges/AttackRangeHysteresis-20261025. 플레이 확인: 85/100m 경계 왕복·일반 NPC 사격 재개·무기 교체에 따른 거리 갱신·보스 충전 중 추격·시야/사선 차단·사망/트리 종료 회귀

몸 정렬 조건을 Attack 진입 전이에 추가하지 않음. 사거리·사선 확보 후 Attack에서 제자리 회전 대기. Idle의 포괄 기억→Chase가 있으므로 초기 안내의 별도 Idle 사선 전이는 중복 제거 기준

## 공격과 Ready 복귀

`NPC Combat Attack`이 현재 타겟 Focus·몸 방향·무기 Ready·태그·쿨다운·입력 소비·Upper 복귀를 확인하고 의미 입력 발신. GA 직접 호출 없음

- 수납 중에는 Draw만 요청, Ready 대기. 두뇌가 준비시킨 NPC 공격 Task에서 Draw 중복 실행 없음
- `Attack Request Retry Delay` 기본 0.5초는 거부·미소비 요청 복구용. 발사 간격 조절값 아님
- Upper Ready Task가 손 부착 성공 후 `PublishUpperState(ReadyEntered, Source=EntityCharacter)` 발행. 정상 경로 Running 유지, 부착 실패만 Task 실패
- Attack 완료·실패 → Ready, Reload 완료·실패 → Ready. CarryState=Ready만으로 ReadyEntered 알림을 대체하지 않음
- 활성 GA의 `State.Combat.Attacking`과 실행 완료 이벤트 수명 일치. 시작 성공 직후 GA 종료·매 Tick 재활성화 금지

### Upper 공격 Task 노드

NPC 전용 이름 `STT_NPC_Attack`은 과거 연결안. 동일 계약의 공통 Task를 재사용할 수 있으며 실제 트리의 Task를 기준으로 확인. `bAbilityStarted=false` 기본값, 부모 EntityCharacter·EntityContext 바인딩 재사용

```text
EnterState
→ bAbilityStarted=false
→ Publish Upper State
   Target=Context.StateEventBus, Event=FireEntered, Source=EntityCharacter
→ Try Activate Ability By Tag
   Target=Context.StateMediator, Ability Tag=Ability.Weapon.Fire
→ Branch(Return Value)
   ├─ false → 구성·활성화 거부 원인 확인 → FinishTask(false)
   └─ true → bAbilityStarted=true → Running

Tick
→ bAbilityStarted=true일 때 HasMatchingGameplayTag
   Target=Context.AbilitySystem, Tag=State.Combat.Attacking
   ├─ true → Running
   └─ false → bAbilityStarted=false
      → FinishTask(Context.WeaponComp.DidLastFireSucceed)

ExitState
→ bAbilityStarted=true일 때 먼저 false
→ Context.StateMediator.CancelAbilityByTag(Ability.Weapon.Fire)
정상 완료 Exit는 재취소 없음
```

GA의 완료 Event Bind·Commit 검사·실패/취소 정리는 [플레이어 무기 연결](플레이어%20연사·빈%20탄창%20재장전%20연결.md)의 GA_Attack 계약 공통 적용. Player Task의 필요 시 Draw 대기와 NPC 두뇌의 준비 경로는 구분

## 몸 정렬과 총구 사선

| 항목 | 현재 기준 |
|---|---|
| 몸 회전 | NPCCharacter.FaceRotation. 타겟/기억 방향을 DA→ASC→CharacterMovement의 공통 RotationSpeed로 추종 |
| 몸 완료 | IsFacingCombatTarget. Attack Facing Tolerance Yaw 기본 5도 |
| 원거리 준비 | 몸 정렬 + 손 부착 + 무기 정의 + 실제 Muzzle Transform 존재 |
| 원거리 발사 공간 | HasClearFiringLine. 몸통 총구 높이→Muzzle→가시 타겟을 max(몸 Capsule 반경, 실제 탄 반경) Sphere Sweep. 물리 탄은 실제 진영 채널, Hitscan/Beam은 무기 DA AttackTraceChannel(기본 WeaponAttack) |
| 장애물 | 자신 제외, 첫 Blocking Hit가 타겟이면 허용. 낮은 엄폐물도 채널을 Block해야 차단 |
| 실행 안전 검사 | AI 요청·FireEquippedWeapon·몽타주 시작·Weapon_FireCommit에서 재검사 |

- **Muzzle 로컬 +X를 몸 회전·발사 허용 기준으로 사용하지 않음.** 소켓 축 보상으로 90도 측면 사격을 만들었던 이전 변경 제거. `Attack Muzzle Tolerance Yaw`도 제거
- IsWeaponFacingCombatTarget은 몸 정렬·손 부착·Muzzle 존재 검사. MuzzleYaw는 로그 진단값이며 허용 기준 아님
- 정렬 이탈은 Attack에서 발사 취소·재정렬, 사선 차단은 입력/GA/발사 취소 후 Chase. 거부된 Commit은 추가 탄 생성·소비 없음
- Chase에서 가시 타겟이 있으면 Draw·무기 활동 유지. 사선 확보 후 Attack 전이. 경로 끝점 차단 시 기존 제한 재시도, 순간이동·강제 발사·별도 엄폐/횡이동 탐색 없음
- 타겟 상실 시 과거 Aim Solution 제거, Controller Pitch/Roll=0, Yaw는 몸 방향 유지. 유효 Aim Solution에서 추가 척추 보정을 재활성화하지 않음
- 공통 상체 보정 회귀는 기존 Entity Tick 분기 복구 후 Player/NPC 자세 회복 사용자 확인. PT-00-3 패스 유지

발사 간격은 무기 AttackCooldown. 현재 DA_BaseAR 저장값 0.1초. 실제 Commit 전 NextAllowedFireTime·Timer 선점, 실패 시 복원, 취소·Ready 복귀로 환급 없음. RapidFire 몽타주 후반 정리·실제 Held 반동 계약은 [무기](../03_시스템%20설계/무기.md)와 플레이어 연결 문서 참조. 과거 1초 튜닝 기록은 현재 DA_BaseAR 값과 구분

## 추적과 기억

### 동작·설정

| 항목 | 기준 |
|---|---|
| 위치 기억 갱신 | Sight + LineOfSightTo 통과 시에만 갱신. 벽 뒤 Actor의 실제 이동 위치 추종 금지 |
| 타겟 상실 | 공격·Actor Focus·기존 이동 중지, 마지막 확인 위치 유지 |
| LostTargetMoveDelay | 상실 후 출발 전 대기. 기본 0, DA_NPC_Humanoid Combat Tendency에서 설정 |
| GiveUpTime | 마지막 위치 접근 완료 또는 경로 실패 복구 후 탐색 대기. 타겟 상실 순간부터 세지 않음. 별도 SearchTime 필드 없음 |
| ChaseRetryDelay / ChaseMaxRetries | 경로 실패 재시도. 기본 0.5초 / 추가 3회 |
| 가시 타겟 도착 반경 | min(MoveAcceptanceRadius, EffectiveRange × ChaseAttackRangeFraction) |
| 기억 위치 도착 반경 | MoveAcceptanceRadius |

NavMesh 밖·분리된 섬은 부분 경로로 도달 가능한 끝점 산출 후 실제 완전 경로 요청. 세계 전체의 최근접 지점 보장은 제외. 도착 후 배회 이동 없이 GiveUpTime 재감지 대기. 재발견 시 이전 이동·상실/탐색 대기 정리 후 Visible 복귀

### STT_NPC_ChaseWalk 노드

Faction 후속 기준은 [대상·공격 필터](../03_시스템%20설계/NPC%20행동.md#faction-기준-대상공격-필터--2026-10-26) 참조. NPC 타겟은 조종자 종류와 무관하게 Allies↔Enemy, 근접 접근/타격/차폐도 아군 제외. 플레이어/우호 NPC Allies·적 NPC Enemy DA 설정과 일괄 빌드 후 검증 필요. Task에 별도 진영 분기를 추가하지 않음.

근접 좀비의 Chase→Attack 정지 틈 수정은 [시스템 설계의 이동 인계](../03_시스템%20설계/NPC%20행동.md#근접-추격공격-이동-인계--2026-10-26) 기준. 아래 Task API는 유지하되 Controller가 공격 접근 인수 전 ChaseCompleted를 지연하고 실제 인수 때 속도를 보존해 경로 해제. 실제 타격 거리에 도달할 때만 멈춰야 함. 빌드·PIE 확인 전.

기존 부모 Pawn 참조에서 Controller 확보. `NPCController`: AstraNPCAIController Object Reference. Task는 완료 판정 참여·Running 유지. 기존 BP AI MoveTo·사거리×0.2 계산·Exit StopMovement 중복 실행 제거

```text
EnterState → 바인딩 Pawn.GetController → Cast To AstraNPCAIController
├─ 실패 → 구성 오류 로그 → FinishTask(false)
└─ 성공 → NPCController 저장 → RequestChaseMove 한 번 → Running

Tick → NPCController.UpdateChaseMove → Running
   Waiting / Moving / Reached / Failed / Idle은 진단 출력용
   도착·재시도 소진 자체로 FinishTask 호출 없음

ExitState → Controller 생명주기에 맞춰 CancelChaseMove → 참조 해제
```

Controller가 목적지·PathFollowing·도착·제한 재시도·탐색 만료를 소유. Task의 LastKnownLocation 저장·매 Tick RequestChaseMove 재발신 없음. 취소는 이동 요청만 정리하며 타겟 기억·탐색 시간은 유지. 사망·UnPossess·Controller 종료는 ForgetCurrentTarget으로 구독·기억까지 정리

## 순찰과 피격 방향

### DA 설정

기존 EntityDataAsset의 Combat Tendency 사용. 별도 DA 추가 없음

| 필드 | C++ 기본값 | 의미 |
|---|---|---|
| bEnableIdlePatrol / PatrolRadius | false / 0 | 순찰 활성과 스폰 위치 중심 반경. 순찰 사용 시 양수 반경 필요 |
| PatrolWaitTime / PatrolAcceptanceRadius | 0 / 50 | 도착 후 대기 / 도착 허용 거리 |
| PatrolSelectionAttempts | 5 | 한 번의 목적지 선택에서 경로 검사 횟수 |
| DamageLookWaitTime | 0 | 피격 방향 회전 완료 후 대기 |
| DamageLookRotationRate / DamageLookToleranceYaw | 360도/초 / 5도 | 회전 속도 / 완료 오차 |

순찰 중심은 최초 Possess 위치. 같은 Pawn 재Possess·Idle 복귀로 재설정하지 않음. 반경 내 도달 가능한 NavMesh 목적지·완전 경로 선택, 반경 안에서 시작한 경로는 경로 점도 반경 내 제한. 전투로 벗어난 경우 반경 내 복귀 허용. 선택/이동 실패는 ChaseRetryDelay 뒤 재시도

### STT_NPC_Patrol

```text
Brain Idle → NPC Idle Command
├─ Enter/Tick.UpdateIdleWeapon → 자동 수납 타이머 갱신
└─ PatrolStarted → NPCCharacter → Mediator.Patrol → Base Patrol
   Enter → Pawn.GetController → Cast → 저장 → RequestIdlePatrol
   Tick  → UpdateIdlePatrol → Running
   Exit  → CancelIdlePatrol → 참조 해제
Brain Idle Exit → PatrolCompleted → CancelPatrol
```

순찰 Task 완료 판정 참여. Waiting/Moving/Failed를 즉시 Task 완료로 사용하지 않음. 별도 BP 난수 목적지·AI MoveTo·Timer 중복 없음. Idle·Patrol 진입에서 Holster/Draw·Montage·강제 부착 요청 없음. Idle의 무기 활동 갱신으로 기존 휴대 상태 유지

### STT_NPC_DamageLook

Base HitReaction의 기존 Task·전이 활용. 이미 배치된 상태를 신규 생성 대기로 기록하지 않음. 회전 Task 실행 중만 추가 공격 요청 대기, 피격 방향 정보 대기만으로 발사 차단하지 않음

```text
피해 → Health.OnDamageTaken → Mediator의 Upper 행동 취소·방향 중계
→ NPCCharacter → Controller.RememberDamageDirection → Base HitReaction

Enter → 부모 Pawn에서 Controller 확보·저장 → BeginDamageLook
├─ true → Running
└─ false → 방향 정보 없음 → FinishTask(true)
Tick → UpdateDamageLook(DeltaTime=Task Tick DeltaTime)
├─ true → FinishTask(true)
└─ false → Running
Exit → Controller 생명주기에 맞춰 EndDamageLook → 참조 해제
```

피격 순간 공격자 Instigator 위치, 없으면 DamageSource 위치 저장. Source 누락·자신·동일 위치 피해는 회전 생략. 시야 감지 전 CurrentTarget·위치 기억 생성 없음, 회전 중 공격자 실제 위치 재조회 없음

완료 후 현재 BrainState 기준: Idle→Patrol, Chase→ChaseWalk, Attack→Base Idle, 사망→Death. HitReaction 자기 재진입·Chase/Patrol 즉시 이탈로 최신 방향 요청 소실 방지. 피격 몽타주 Task 병용 시 All 완료 정책. 실행 참조 누락은 오류·Task 실패 복구, 정상 완료와 구분

## 착지 후 현재 의도 복구

BrainState는 의도, Base는 현재 몸 실행 상태. Falling 진입에서 이동 요청만 취소하며 BrainState·타겟 기억 초기화 없음. 착지는 이전 Base 상태 복원이 아니라 **현재 BrainState** 재평가

```text
GroundedEntered → EventBus → Mediator.OnGroundedReady
→ NPCCharacter → Controller.ReissueCurrentLocomotionIntent
```

| 현재 의도 | 재발행 |
|---|---|
| Idle + 순찰 사용·양수 반경 | Patrol |
| Idle + 순찰 비활성·반경 0 | Stop |
| Chase + 타겟 기억 | Chase |
| Attack·기타·기억 없는 Chase | Stop. 기존 Brain 공격 Task 유지 |

Ground 공통 알림 Task는 GroundState 부모에 한 번 배치. 실제 에셋명 STT_NPC_GroundBase, 초기 안내명 GroundReady. 자식 Idle/Patrol/ChaseWalk마다 중복 배치 없음

- Should State Change on Reselect 해제: 자식 전이마다 알림 재발행 방지
- Considered for Completion 해제: 첫 Tick 성공 완료가 Ground 부모를 종료하지 않도록 설정
- Should Call Tick Only on Events 해제: 최초 Tick 확보
- Enter는 Running. 첫 Tick에서 `PublishLocomotionState(GroundedEntered, Source=Self, Target=EntityContext.StateEventBus)` → FinishTask(true). EventBus 누락은 구성 오류 출력·실패
- Enter 대신 첫 Tick은 초기 StartLogic보다 늦은 NPC 구독 시점 대응. 별도 발행 Bool·의도 감시 Tick 전이 없음
- Exit는 추가 실행 없음. 첫 Tick 전 이탈은 알림 미발행, 다음 Ground 진입에서 새 알림

Falling 접지 목적지는 GroundState, 기본 자식 Idle. 부모의 `StateTree.Event.LocomotionCommand` + `FLocomotionStateCommandPayload.Command` 전이: Patrol→Patrol, Chase→ChaseWalk, Stop→Idle. 자식의 동일 명령 전이로 가리지 않음. CancelPatrol/CancelChase는 해당 이동 상태로 범위 제한, CancelSprint는 Sprint에만 적용. Death·Ground→Falling 유지

이동 Task Exit의 취소는 유지. 착지로 GA 재활성화·쿨다운 초기화·정확한 과거 순찰 진행률 복원 없음. 순찰은 목적지·대기 재선택, Chase는 이동 재요청

## 정상·실패·취소 전체 흐름

```text
Root 추적 갱신 → Idle의 순찰·무기 활동 유지
→ 가시 타겟 발견·위치 기억
   ├─ 사거리 밖 / 사선 차단 → Brain Chase → Base ChaseWalk
   │  → RequestChaseMove 한 번 → Tick.UpdateChaseMove
   │     ├─ 사거리 안 + 사선 확보 → Attack → 이동 취소
   │     └─ 경로 실패 → 제한 재시도 → 탐색/실패 대기, 발사 금지
   └─ 사거리 안 + 사선 확보 → Attack
      → NPC Combat Attack 지속 → 무기 Ready·몸 정렬 대기
      → 의미 입력 → Upper Task → GA 한 번 활성화
         ├─ 거부 → Task 실패 → ReadyEntered → 재요청 판단
         └─ 시작 → Fire Montage → Commit 정렬·사선·쿨다운 재검사
            ├─ 유효 → 공격 확정 → 완료 Event → GA 종료
            │  → Task 결과 → ReadyEntered → 다음 요청 판단
            └─ 무효 → 탄 생성·소비 없음 → 발사 취소

시야 상실 / 타겟 Actor 소멸
→ 공격·Actor Focus·기존 이동 정리 → 위치 기억 유지
→ LostDelay → LastKnownMove → Searching(GiveUpTime)
   ├─ 재발견 → 이전 요청 정리 → Visible → Chase/Attack
   └─ 만료 → 기억 제거 → Idle / 순찰

낙하 → 이동 Task Exit 취소, 현재 의도 유지
→ Ground 첫 Tick 알림 → 최신 의도로 Patrol/Chase/Stop 재발행
피격 → 이동 취소 → DamageLook → 완료 후 현재 의도로 복귀
사망 / UnPossess / 트리·Controller 종료
→ 공격·이동·회전·Focus·구독·기억 정리 → 공통 Death 또는 종료
```

## 회귀 확인과 진단 기록

| 장면 | 확인 기준 |
|---|---|
| 뒤/옆 타겟·급이동 | 몸 정면 허용각 안에서 발사, 소켓 축으로 몸 90도 보상 없음 |
| 낮은 엄폐물·Commit 직전 차단 | 발사 금지·추가 소비 없음, Chase 이동 후 사선 확보 시 공격 |
| 연속 발사·취소·재발견 | Owner별 실제 발사 간격 ≥ 현재 무기 AttackCooldown. 취소로 제한 초기화 없음 |
| 기억 추적 | 벽 뒤 실제 Actor 위치 추종 없음. 지연·기억 이동·도착/실패 후 GiveUpTime 적용 |
| 분리 NavMesh·경로 실패 | 부분 경로 끝점·제한 재시도, 매 프레임 MoveTo 재발신 없음 |
| 순찰·피격 | 스폰 중심 유지·도착 대기. 피격 순간 좌표 회전, 시야 전 타겟 획득 없음 |
| 낙하·재착지 | 최신 BrainState 복구. Ground 알림 반복·추가 발사·부모 조기 종료 없음 |
| 탄창 소진·GA 거부 | 기존 Reload 정상 완료·Ready 복귀, 매 Tick GA 재활성화 없음 |
| 사망·종료 | 이동·회전·GA·구독 정리. 파괴된 Controller에 종료 API 호출 없음 |

`astra.NPC.CombatDebug 1` 사용, 종료 후 0으로 해제. 여러 Owner의 발사 간격 합산 금지

| 로그·API | 용도 |
|---|---|
| NPCCombat Reason | DamageLookActive / TagOrActionBlocked / WeaponNotReady / FacingTarget / AwaitUpperReady / BufferedInputUnconsumed / WeaponCannotAttack / AttackRequested 등 대기 원인 |
| Facing·MuzzleFacing·ClearFiringLine·BufferedAttack | 몸·준비·사선·입력 확인. Facing=1, MuzzleFacing=0이면 손 부착·무기 정의·총구 존재 확인 |
| BodyYaw·MuzzleYaw·ViewPitch·Focus | 회전·시선 진단. MuzzleYaw는 발사 허용 기준 아님 |
| FiringLineBlocked / ChaseForFiringLine | 차단 전이 바인딩·Collision 채널·이동 대기 |
| WeaponShot Time·Interval·Cooldown·NextAllowed | 실제 발사 간격. 첫 발 Interval=-1 |
| GetTargetTrackingPhase·HasVisibleTarget·HasTargetMemory·GetLastKnownTargetLocation·GetChaseMoveStatus·GetChaseDestination | Visible/LostDelay/LastKnownMove/Searching·이동 상태 구분 |

### 보존할 확인 결과와 별도 잔여

- 10/21 추적·재장전 완료 사용자 확인. 사망 전이 해결·Critical 우선권, PT-00-3 패스 유지. 재현 보고 없는 동일 진단 재개 없음
- 10/20 추적 진단에서 Ground 알림 Task의 완료 판정 참여, 사선 Invert 반대 조합을 지적. 현재 전이표·Ground 설정에 정정 내용 통합. 단일 수정만이 완료 원인이었다고 사후 확정하지 않음
- 캐릭터 위 Debug Text 소멸은 StateTree 실행 정지 증거 아님. 실제 활성 Brain·Base 상태로 판단. Chase TasksCompletion=All에서 모든 Task 완료 판정 제외만으로 즉시 완료 단정도 금지
- 구 백업 `Saved/Logs/Astra_SoulLink-backup-2026.10.20-07.30.03.log`: NPC DA_BaseAR 19발, 연속 18간격 0.166668~0.350003초, 0.1초 미만 0건. Player 발사·반동 검증 근거와 구분
- 해당 구 로그의 발사 직후 ClearFiringLine=0 반복: 실제 Hit Actor 미확정. 자기 Projectile 차단은 가설, 해결 완료/확정 원인으로 기록하지 않음
- 10/20 UTC 11:34:42 PIE BeginTearingDown 직후 STT_NPC_DamageLook.EndDamageLook의 무효 NPCController 접근 기록. 재발 여부 미확인. 종료 정리 경로의 별도 잔여, 완료된 추적·재장전의 미완료 근거로 재사용하지 않음
- 같은 날 PropertyEditor DetailLayoutHelpers.cpp:138 Ensure와 총기 변경의 인과 미확정. 변경 후 바이너리·모든 무기 조합 검증을 과거 빌드나 구 로그로 대체하지 않음

## 충전 유탄·착탄 경고 연결 — 2026-10-24

상태: C++ 반영·소스 정적 검토. 이번 변경의 빌드·UHT·Editor·Automation·PIE 미수행, uasset 변경 없음

최신 사용자 확정: 보스는 항상 100% 충전 후 자동 발사·BaseDamage ×1.5. 플레이어의 부분 충전 해제 규칙과 구분. 기존 무소모·고정 탄속·ArcHeight·접촉 즉시 폭발 유지

### Editor 연결

| 위치 | 설정 |
|---|---|
| 보스 무기 DA Shoot Type | ChargedShot |
| Attack Kind / Delivery Type | Ranged / PhysicalProjectile 유지 |
| Uses Ammo | false 유지 |
| Non Ammo Projectile Data | 기존 정상 발사되는 유탄 DA 유지 |
| Weapon Charge / Auto Fire When Fully Charged | **보스는 true 필수**. 새 기본값 false는 플레이어 입력 해제형 |
| Weapon Charge / Min Charge Ratio | 0.2. 플레이어는 이 비율 이하에서 해제하면 미발사 |
| Weapon Charge / Partial Charge Damage Multiplier | 1 |
| Weapon Charge / Full Charge Damage Multiplier | 1.5. 보스는 이 배율로 발사 |
| Weapon Charge / Max Charge Time | 사용자 지정 충전 시간. 도달 시 자동으로 Fire Montage 시작 |
| Weapon Charge / Visual / Charge Visual Type | **보스 구체 성장은 GrowingProjectile 지정**. 기본값 None은 충전 연출 없음 |
| Weapon Charge / Visual / Charge Start Scale | 성장 연출의 충전 시작 크기 배율. 기본 0 |
| Weapon Charge / Visual / Charge End Scale | 성장 연출의 충전 완료·발사 후 메시 크기 배율. 기본 1, 원래 메시 크기 기준 |
| ABP_Boss / FireMontages[GrenadeLauncher] | 기존 Boss_Fire_Montage 유지. 연결된 Slot과 Weapon_FireCommit Notify 유지 |
| 유탄 DA Projectile Mesh | 성장시킬 실제 구체 메시. 전기 표현은 해당 메시/발사체 BP의 머티리얼로 설정 |
| 유탄 DA Impact Warning Material | 아래 원형 Deferred Decal 머티리얼/인스턴스 지정. None이면 경고 생략 |
| 유탄 DA Explosion Radius | 실제 폭발 반경과 경고 반경 공통 값 |
| 유탄 DA Aim At Target Ground | true 유지 |
| 유탄 DA Speed / ArcHeight | 기존 사용자 설정 유지 |

기존 MinChargeHoldTime·Min/MaxChargeDamageMultiplier는 비노출 로딩용으로 유지. 새 MinChargeRatio·Partial/FullChargeDamageMultiplier 설정 사용. 구버전 MaxChargeDamageMultiplier=3이 완충 피해에 적용되지 않음

충전 중 주기적 탄약 소비는 이번 범위 밖. 기존 ChargeAmmoConsumeInterval·ChargeAmmoPerTick 선언과 실제 구현을 구분. 현재 탄약 사용 무기는 발사 성공 때 AmmoPerShot 소비, 보스 무소모 유지

공통 충전의 다른 조합:

| 예시 | 설정과 수명 |
|---|---|
| 단발 유탄 | WeaponType=GrenadeLauncher, ShootType=SingleShot. 충전 준비 없이 기존 곡선 발사·착탄 경고 |
| 플레이어 충전 유탄 | ShootType=ChargedShot, AutoFire=false. 20% 이하 해제 미발사, 부분 ×1, 완충 ×1.5 |
| 충전 라이플·이펙트 | ShootType=ChargedShot, ChargeVisualType=WeaponEffect, ChargeVFX 지정. 충전 중 실제 탄 생성 없음 |
| 충전 Hitscan/Beam | 해당 DeliveryType 유지, None 또는 WeaponEffect 연출. GrowingProjectile은 PhysicalProjectile 전용 |

WeaponEffect는 무기 DA의 ChargeEffectSocketName에 부착. None이면 무기 메시 원점. ChargeProgressParameterName은 선택적 Niagara float 이름이며 지정한 경우 0..1 비율 전달. 발사/취소/교체/월드 종료에서 제거. 별도 무기 고유 능력 GA 추가 없음

원형 데칼 기본 노드:

```text
Material Domain = Deferred Decal
Blend Mode = Translucent
색상 파라미터 ───────────────────────────────────────────→ Emissive Color
TextureCoordinate(RG) → Subtract(Constant2Vector 0.5,0.5) → Length → If → Opacity
                                                                B = 0.5
                                                                A < B = 1
                                                                A = B = 0
                                                                A > B = 0
```

UV 중심 0.5와 반경 0.5는 데칼 영역에 원을 맞추는 기하 기준. 색상·투명도·테두리 표현은 사용자 머티리얼 설정. 지면 메시의 Receives Decals 활성화 필요. 현재는 고정 원형 영역이며 착탄까지 채움/확대 애니메이션 없음

### 정상·실패·취소 순서

```text
기존 Attack Task → GA → FireEquippedWeapon
├─ 충전 정의/머즐/선택한 연출 준비 실패 → false → 기존 GA/Task 실패 처리, 경고 없음
└─ 충전 시작 → 기존 무기 갱신, 선택한 연출만 실행
   ├─ 시야/사선 상실·취소·사망·무기 교체 → 준비 탄/이펙트 정리 → 기존 실행 종료
   └─ 발사 시점
      ├─ 보스 AutoFire=true → 100%에서 자동 요청, ×1.5 고정
      └─ 플레이어 AutoFire=false → 입력 해제
         ├─ 20% 이하 → 미발사·연출 정리·실행 종료
         └─ 부분 ×1 / 완충 ×1.5 고정
      → Fire Montage
      ├─ 몽타주 실패·중단·Notify 없음 → 연출/준비 탄 정리 → 기존 실행 실패/종료
      └─ Weapon_FireCommit → 최신 목표·전달 방식 확정
         ├─ 발사 실패 → 연출/준비 탄 정리, 경고 없음
         └─ 성공 → 고정한 피해 배율·쿨다운 시작·충전 이펙트 종료
            → 유탄은 고정 지점 경고 표시, 성장 연출은 같은 준비 탄 발사
            ├─ Pawn/지형 충돌 → 폭발 → 풀 반환·경고 제거
            ├─ 수명/곡선 종료 → 풀 반환·경고 제거
            └─ 직접 제거/월드 종료 → 경고 제거
발사 후 공격 Sequence 종료·다음 충전 취소 → 이미 비행 중인 탄은 유지
```

사용자 플레이 확인 항목: 보스 완충 자동 발사·피해 ×1.5, 충전 시간·머즐 부착/성장·발사 후 최종 크기, 실제 발사 순간의 고정 경고, 이동한 타겟을 따라가지 않음, 조기 충돌·취소·사망·무기 교체 정리, 2개 보스 동시 충전의 독립성, 일반 NPC SingleShot/RapidFire 회귀. 플레이어는 20% 경계·부분/완충 피해·99% 해제 후 몽타주 대기 중 배율 유지·100% 유지 후 해제·Draw 전 해제 확인. None/WeaponEffect에서 준비 탄 없음, Beam/Hitscan 조합도 별도 실행 확인 대상. 정적 검토로 실행 통과를 주장하지 않음

### 보스 추격 별도 확인

이전 보고: 보스 Patrol·인식 시 발사 정상, 추격 불가. 추격은 별도 Chase AbilitySpec 없이 Brain Chase 명령→Base ChaseWalk→Controller의 RequestChaseMove/UpdateChaseMove 사용. 후속 사용자 확인에서 GroundState→ChaseWalk 우선순위 조정 후 추격 이동됨. 최신 이슈는 추격 범위 안에서 타겟 이동에 따른 걷기 끊김이며 위 분리 절에서 다룸

## 발사체 충돌·지속 이펙트 — 2026-10-24

### 2026-10-25 재발 보고·추가 보호

- 사용자: 비행 탄 교차·충전 준비 탄 피격 두 경우 모두 소멸 재발, Niagara는 다른 이펙트로 처리. 충돌 이슈 해결 지정 취소
- 현재 Native OnHit는 이미 AProjectileActor 상대를 제외하므로 프로필/실제 클래스/다른 BP Hit·Destroy/시각 효과 소멸 경로 구분 필요. 실제 PIE 원인은 미확정
- 추가 반영: 준비/발사 진입에서 모든 AProjectileActor를 양쪽 이동 Sweep Ignore Actor에 등록. 프로필 외에 Actor 단위 제외, EndPlay에서 상대 Ignore 참조 정리. 프레임 Tick 전수 탐색 없음
- BP InitialLifeSpan Destroy 타이머는 준비/회수/발사 진입에서 해제해 DA/Factory 수명과 분리. BP의 Delay→Destroy나 개별 Niagara Kill 설정은 이 변경으로 확인·제거한 것이 아님
- 중앙 진단 astra.Debug Weapon.Projectile 1 추가. Prepared/Launched/Hit/무시된 Hit/LifetimeExpired/CurveEnd/Deactivated/EndPlay와 실제 충돌 프로필·채널·상대 클래스/컴포넌트·Actor LifeSpan·활성 충돌 컴포넌트 기록
- 이 로그로 다음 재현의 원인 판정 필요. 이번 보강은 소스 정적 검토까지, 빌드·UHT·Editor·Automation·PIE 미수행. 원인 확정·실제 재발 해소 확인과 구분
- 백업 Saved/CodexChanges/ManualRecoilProjectile-20261025-011810

#### 사용자 재현 로그로 확인한 충전 구체 회수 원인

- 플레이어 BP_TestBulletActor의 HitActor는 BP_BossCharacter_C_5, HitComponent는 CollisionCylinder, HitProfile은 Pawn. 충전 구체를 맞힌 기록이 아님
- 충전 BP_TestBossBullet은 Prepared 시 ActorCollision=0, CollisionParts 비어 있음. 보스 본체 Hit 직후 Active=0·Prepared=1 상태의 Deactivated가 반복되고 STT_NPC_DamageLook의 TakeDamage가 뒤따름
- 같은 BP_TestBossBullet_C_0이 다시 Prepared로 재사용됨. 로그 마지막 EndPlayReason=2는 PIE 종료 시점. 해당 재현은 탄끼리 접촉 파괴가 아니라 본체 피격에 따른 충전 취소·풀 회수로 판정
- 코드 대조: Health.OnDamageTaken → Mediator.HandleDamageTaken이 Upper.CancelAction·피격 방향 이벤트·Lower.HitReaction 전달. Controller.RememberDamageDirection도 전투 중 StopCombatFire(DamageLookRequested)를 호출. 취소된 Sequence의 ClearChargeState → Factory.CancelChargedProjectile → ReleaseProjectile로 준비 구체 회수
- 사용자 확정: 보스는 일반 피격에도 충전 유지, 사망 시 취소. 경직·넉백·피격 몽타주·행동별 중단 조건의 전체 기획은 미정이며 이번 변경으로 확정하지 않음
- 이번 로그는 충전 중 사례의 원인 증거. 별도 비행 탄 교차 사례 해결이나 모든 피격/취소 회귀 완료로 확대하지 않음. 이 후속 진단에서 C++ 수정·빌드·PIE 실행 없음

#### 보스 일반 피격의 행동 중단 정책 — 2026-10-25

최신 사용자 확정: 보스의 일반 피격은 체력만 차감하고 회전·이동·충전·발사를 중단하지 않음. 체력 임계치에서 기절시켜 모든 행동을 멈추는 기능은 후속이며 이번에 구현하지 않음. 아래 충전 전용 보호 절은 이전 구현 이력

- Entity DA → Combat | Damage Reaction → `Can Interrupt Actions On Damage`를 보스에서 false. 일반 NPC·Player 기본 true 유지. 무기 DA 옵션 아님
- 최신 C++는 충전 전·발사 후·추격 중 일반 피격의 Upper.CancelAction·Lower.HitReaction을 생략하되 OnDamageDirectionRequested는 항상 전달. NPC 방향 인지와 행동 중단을 분리하며 자세한 정상·실패·취소 흐름은 위 분리 절 참조. Health의 피해·사망 이벤트 경로 및 명시적 취소 변경 없음
- 기존 `bCanInterruptChargedAttackOnDamage`에서 `bCanInterruptActionsOnDamage`로 PropertyRedirect 추가. 저장된 true/false를 그대로 이전하므로 보스 false 값 확인 필요. 이전 Editor 읽기 확인에서는 DA_NPC_DemoBoss 옵션 true였고 이후 사용자가 직접 끔 확인
- 최초 정책 확장 당시 소스/설정 3개 파일 정적 반영 기록. 최신 분리는 위 절의 9개 C++ 파일/정적 검토 기준이며 이번 빌드·PIE 전. 새 Source 테스트 및 uasset 편집 없음. 기절·경직·넉백 로직 추가 없음

```text
일반 피격 → Health에서 HP 차감
├─ HP 0 → State.Dead·OnDeathStarted
│  → 기존 Ability 취소·Upper Disable·Base Death 명령 → 공통 Death Task/GA/연출
└─ 생존 → Mediator.HandleDamageTaken
   ├─ 방향 사실 전달 → 보스 미인지 시 최초 공격자 위치 기억 (시야 획득 아님)
   ├─ CanInterruptActionsOnDamage=false → 기존 행동 유지, DamageLook 요청 없음
   └─ true 또는 Entity DA 없음 → 기존 CancelAction·DamageLook·HitReaction
명시적 취소/무기 교체/기억 만료/트리 중단 → 기존 취소 정책 유지
```

플레이 확인: 보스 옵션 false에서 충전 전·중·발사 후·추격 중 연속 피격 시 HP 감소와 기존 동작 유지, 치명적 피해 시 Death 선점·제거, 일반 NPC 기본 true 반응 회귀. 이 정적 반영을 실행 통과로 기록하지 않음

이전 추격 보고: 피격되지 않은 보스도 발견→Chase에서 이동하지 않음. 후속 사용자 확인에서 GroundState→ChaseWalk 전이 우선순위 조정으로 추격 이동 성공. 피격 정책 변경의 해결 효과로 기록하지 않음

이전 사망 보고: HP 0·BrainState 비전투·Idle 모션 잔류. 후속 사용자 확인에서 Death 전이 Priority=Critical 변경으로 해결. ForgetCurrentTarget의 Idle 값만으로 사망 실패를 판단하지 않음. [공통 Death 연결 기준](공통%20사망%20GA·몽타주·제거%20연결.md)

제공 DamageLook Task에는 정상 Tick 완료의 EndDamageLook만 있고 ExitState 정리가 없었음. 사용자가 아래 중단 경로를 연결할 필요가 있음. 최신 에셋 반영 여부 미확인

```text
Enter → GetAIController → Cast AstraNPCAIController
├─ 실패 → FinishTask(false)
└─ 성공 → NPCController 저장 → BeginDamageLook
   ├─ true → Running
   └─ false → FinishTask(true)
Tick → IsValid(NPCController)
├─ false → FinishTask(false)
└─ true → UpdateDamageLook(DeltaTime=Tick.DeltaTime)
   ├─ false → Running
   └─ true → EndDamageLook → FinishTask(true)
ExitState (정상·실패·중단·사망)
→ IsValid(NPCController)
   ├─ true → EndDamageLook → Set NPCController=None
   └─ false → Set NPCController=None
```

#### 보스 충전 피격 보호 연결 — 2026-10-25

- C++ 추가: Entity DA의 Combat | Damage Reaction → `bCanInterruptChargedAttackOnDamage` (`Can Interrupt Charged Attack On Damage`). 기본 true는 기존 피격 반응 유지
- **보스 BP가 참조하는 Entity DA에서 해당 옵션을 false로 설정**. 무기 DA 옵션이 아님. 일반 NPC·Player DA는 기본 true 유지. uasset은 이번에 수정하지 않음
- false이며 충전 공격 Sequence가 진행 중일 때만 Mediator에서 Upper.CancelAction·피격 방향 이벤트·Lower.HitReaction을 함께 생략. 충전 시작부터 완충 후 발사 몽타주 종료까지 보호하며, 일반 단발/연사·재장전·대기 중 피격 정책은 바꾸지 않음
- 상태 소유는 AttackSequenceHandler의 bCharging/bChargeResolved/bFireSequenceActive. WeaponComponent의 IsChargedAttackInProgress는 읽기 전용 조회. Mediator는 Entity DA 정책과 이 조회만 사용하고 충전·피해를 직접 변경하지 않음
- HP 차감은 이미 Health에서 적용된 뒤이며 피해 무효화가 아님. HandleDeathStarted의 Ability 취소·Upper Disable·Lower Death는 변경하지 않음. 수동 취소·Holster·무기 교체 등 기존 취소 경로도 유지
- 이는 확정된 보스 충전 예외의 구현. 전체 피격 반응 시스템·강인도·경직·넉백 수치와 정책은 미정
- 이번 5개 C++ 파일 정적 검토 범위, 빌드·UHT·Editor·PIE·Automation 미수행. 백업 Saved/CodexChanges/ChargeDamageProtection-20261025

```text
사용자 빌드 → 보스 Entity DA의 Can Interrupt Charged Attack On Damage 끄기 → 저장
└─ 일반 피격 → Health에서 피해 적용
   ├─ 사망 → 기존 Death 경로 → 공격 취소·구체/경고 정리
   ├─ 생존 + 옵션 false + 충전 공격 진행 중
   │  → CancelAction/DamageLook/HitReaction 미발신 → 기존 충전/발사 계속
   └─ 그 외 → 기존 피격 취소·DamageLook·HitReaction 경로
정상: 연속 피격에도 HP 감소·충전 증가·완충 발사, 피격마다 구체 재준비 없음
실패/취소: 사망·수동 취소·무기 교체·준비 탄 외부 유실은 기존 정리 경로 유지
회귀: 일반 NPC/Player 기본 true, 보스 비충전 행동의 기존 피격 반응 유지
```

#### 충전 중 추격과 기억 위치 발사 연결 — 2026-10-25

사용자 확정: 충전 중 이동 가능, 현재 추격 전환에서 공격 취소되는 경로 수정. 타겟이 가려지면 충전 유지 후 마지막 확인 위치로 발사

- C++: Attack Task Exit/시야 상실/사거리 이탈에서 이미 진행 중인 충전만 유지, 새 발사 요청·구독 종료. Attack 재진입도 충전을 재시작하지 않음. Native Root Task Exit와 StateTree 완료/중단은 강제 취소
- NPCCharacter의 진행 중 충전 재검사와 조준은 Controller의 마지막 관측 좌표 사용. 유탄의 bAimAtTargetGround는 해당 좌표 아래 지면 Trace, 직선 충전의 산포는 마지막 관측 타겟 이동 상태 사용. 가려진 Actor의 현재 위치·속도 조회 없음
- 충전은 기존 Entity/Weapon 갱신에서 진행, 새 이동 Tick·충전 Timer·DA 옵션 없음. 캐릭터 이동 속도와 완충 자동 발사 설정 유지, 발사 Commit에서 현재 머즐 기준 궤적/경고 생성
- 사망·UnPossess·기억 만료·전투 상태 이탈·명시적 취소·무기 교체는 강제 종료. 일반 SingleShot/RapidFire의 기존 추격 취소 유지
- C++ 6개 파일 정적 검토 범위. 백업 Saved/CodexChanges/MovingCharge-20261025. 빌드·UHT·Editor·PIE·Automation 미수행, uasset 수정 없음

Editor 확인 순서:

1. 기존 ST_AIBrain Root의 `NPC Update Target Tracking` 유지. Chase에 `NPC Combat Attack`을 추가하지 않음. Chase에서는 기존 충전만 이어지고 새 충전은 Attack 복귀 후 시작
2. Attack의 `NPC Set Brain State=Attack`, Chase의 `NPC Set Brain State=Chase` 및 기존 Chase 이동 명령·Base ChaseWalk 연결 유지
3. Chase 진입 BP에 별도 `CancelFireWeapon`/Fire Ability 취소/Upper.CancelAction을 붙여 둔 경우, 단순 추격 전환용 취소만 제거. 사망·무기 교체·명시적 취소 경로는 유지. Native 수정만으로 별도 BP 취소 노드를 제거한 것은 아님
4. Base ChaseWalk의 Enter=RequestChaseMove, Tick=UpdateChaseMove, Exit=CancelChaseMove는 유지. Upper의 Fire GA/Task가 충전 완료까지 실행 대기하도록 유지
5. 보스 일반 피격도 충전을 유지하려면 앞 절의 보스 Entity DA 옵션 false 설정 필요. 이동 허용과 별개의 피격 예외

```text
Attack에서 충전 시작 → Attack→Chase 상태 전환
├─ 진행 중 ChargedShot: 준비 탄·충전 시간 유지, 하체 추격 이동 계속
│  ├─ 타겟 보임 → 관측 좌표 갱신
│  ├─ 타겟 가려짐 → 저장 좌표 유지, 숨은 Actor 위치 조회 없음
│  └─ 완충 → 기존 몽타주·Weapon_FireCommit → 현재 머즐에서 발사·고정 경고
├─ 일반 SingleShot/RapidFire → 기존 공격 취소
└─ 기억 만료/사망/트리 종료/명시적 취소/무기 교체 → 보존된 충전도 취소·구체 정리
실패: 머즐/준비 탄 유실·발사 설정 오류 → 기존 실패 종료·GA/Task 정리
```

플레이 확인: 충전 중 실제 위치 변화·0% 재시작 없음, Attack↔Chase 왕복 시 구체 중복 없음, 가려진 뒤 플레이어가 이동해도 저장 위치로 발사, 재감지 시 갱신, 이동한 머즐에서 생성·착탄 경고 일치, 사망/기억 만료/트리 중단/무기 교체 정리, 일반 NPC 연사 및 Player 충전 걷기 회귀. 이 변경으로 새로운 보스 이동 패턴을 구현했다고 보지 않음

```text
astra.Debug Weapon.Projectile 1 → 비행 탄 교차 / 충전 준비 탄 피격 재현
├─ Hit → Deactivated: HitActor/HitClass/HitComponent로 실제 차단 대상 확인
├─ IgnoredProjectileHit: Native 접촉 폭발은 실행하지 않음, 이후 BP Destroy 여부 확인
├─ LifetimeExpired 또는 CurveEnd: 충돌 외 정상 회수 경로 확인
├─ Hit/회수 없이 EndPlayReason=0: BP 등 외부 Destroy 경로 확인
└─ Actor 종료 없이 외형만 소멸: Niagara 수명·Collision/Kill·렌더링 확인
정상: 탄끼리 통과, Pawn/지형 접촉 폭발·피해·착탄 경고 정리 유지
실패/취소: 준비 탄 유실 → 기존 공격 실패 종료 → GA/Task 종료·Ready 복귀 확인
```

사용자 확인: 앞선 변경 빌드 완료, 충전 구체 성장 정상. 새 보고: 탄끼리 충돌, 충전 준비 탄이 피격되어 사라진 뒤 공격 진행 불가. 실제 충돌 컴포넌트·BP 피격 그래프는 미확인

### 이번 C++·Config 반영

- Projectile 객체 채널과 AstraProjectile 프로필 추가. 발사체끼리 Ignore, Visibility/Camera Trace에도 Ignore. 지형·Pawn 충돌은 유지
- Native에서 초기화/풀 회수/준비 시 Actor 전체 충돌 끔, 비행은 Sphere만 켬. BP의 메시 충돌 설정이 남아 있어도 Native 수명 진입 시 시각 컴포넌트 충돌 해제
- 발사체 TakeDamage에서 Damage 이벤트 전달 차단, 직접/폭발 피해도 다른 AProjectileActor 제외. 현재 탄을 사격으로 파괴하는 기능 없음
- 준비 탄의 외부 제거는 충전/완충 후 몽타주 구간 모두 감지. 기존 CancelFire와 OnWeaponFireFinished(false)로 종료, 실제 실패 처리·Ready 복귀는 사용자 GA/Task 계약 확인 대상
- 충돌 수정과 Niagara 부착은 소스 정적 검토까지. 이번 변경 빌드·UHT·Editor·Automation·PIE 미수행, 신규 테스트 파일·uasset 변경 없음

### Editor 연결·검증

1. Config에 새 충돌 채널·프로필을 추가했으므로 Editor 재시작 후 확인. Project Settings→Collision에 Projectile 객체 채널·AstraProjectile 프로필 표시
2. BP_TestBulletActor의 메시를 충돌체로 다시 켜는 이벤트/노드 중복 없음 확인. Native의 Sphere가 게임플레이 충돌 담당
3. 유탄 DA Ammunition→Visual→Projectile VFX에 지속 본체용 Niagara 지정. 성장 충전부터 비행까지 유지, 메시 크기 상속. 같은 시스템을 Trail VFX에도 중복 지정하면 이펙트 중복 생성
4. Trail VFX는 실제 발사 이후의 별도 궤적이 필요할 때만 지정. None이면 생략
5. 보스 충전 중 일반 총탄을 통과시켜 준비 탄 유지, 비행 탄끼리 통과 확인. Pawn/벽/바닥 명중의 기존 폭발·피해·경고 정리도 회귀 확인
6. 준비 탄을 외부에서 강제 제거한 경우 공격 종료 실패 알림→GA 종료→Task 실패 완료/Ready 복귀 확인. 기존 GA가 성공 결과만 처리하면 해당 실패 분기도 연결

```text
충전 준비 → Actor 전체 충돌 없음·피격 Damage 이벤트 없음
├─ ProjectileVFX 지정 → 본체 Niagara 생성, 메시 크기 상속
├─ 미지정/생성 실패 → 시각 효과만 생략, 공격 유지
├─ 외부 강제 제거 → 기존 실행 취소 → OnWeaponFireFinished(false)
│  → GA 종료 → Task 실패 완료 → Ready 복귀
└─ 정상 발사 → 같은 본체 Niagara 유지, TrailVFX 선택 생성
   ├─ 다른 발사체 → 충돌 무시·통과
   ├─ Pawn/지형 → 기존 접촉 폭발 → 풀 회수 → 본체/궤적/경고 정리
   └─ 수명/곡선 종료·직접 제거 → 기존 회수/제거 → 본체/궤적/경고 정리
```

### Niagara 0.3초 형태 유지

대상 `/Game/Free_Spells/VFX_Niagara/NS_Free_Spells_Buff_Lightning1`. 사용자 제공 텍스트는 Overview Node 4개의 에미터 핸들·배치 정보이며 Emitter State·Lifetime·크기/알파 모듈 값은 포함하지 않음. 0.3초 이후 사라지는 실제 원인 미확정

사용자 의도: 전기 움직임은 계속 재생, 0.3초 무렵 크기·형태 유지. Niagara 전체 Pause·Desired Age 고정은 사용하지 않음. 원본을 복제한 별도 지속형 콘텐츠에서 확인 권장

| 확인 위치 | 조정 방향 |
|---|---|
| System/Emitter 수명 | 한 번 재생 후 종료하는 구조인지 확인. 필요한 지속 에미터는 반복 설정, Emitter State의 Life Cycle Mode=Self일 때 Loop Behavior=Infinite 설정 가능 |
| Particle Spawn / Initialize Particle | Lifetime 때문에 입자가 끝나는지 확인. Emitter 반복과 개별 입자 수명은 별도 |
| Emitter Update / Spawn 계열 | Burst만 있다면 종료 후 다시 생기는 구조인지 확인. Spawn Rate는 지속 생성 가능하나 기존 효과에 무조건 추가하면 입자 수·밝기 변화 가능 |
| Particle Update / Scale Sprite Size·Scale Mesh Size | 유지하려는 단계의 크기 값을 이후 구간에도 유지. 전체 Particle Age 정지 대신 크기를 제어하는 입력/곡선만 조정 |
| Particle Update / Color·Scale Color | 알파가 0으로 떨어지는 구간 확인·지속 구간 유지. Material의 별도 소멸 파라미터도 확인 |
| 전기 노이즈·UV/Material Time | 움직임을 유지하므로 시간 진행 유지. 형태 유지용 입력과 분리 |
| Local Space | 탄과 함께 이동/성장할 본체 에미터인지 확인. World Space 전제 모듈은 무조건 전환하지 않고 좌표 계산 함께 검토 |

0.3초는 초 단위 재생 위치이며 NormalizedAge=0.3과 같다고 가정하지 않음. 정규화 곡선을 사용하면 실제 Lifetime/Loop Duration에 맞춰 기준 위치 확인. Loop Duration을 0.3으로 줄이는 것만으로 형태 유지가 보장되지 않음

#### 후속 제공 Spark2 판독

사용자 후속 ClipboardContent 기준. Local byte의 float 해석과 설치된 UE 5.8 Niagara UserDefinedEnum의 표시 이름 대조. 에셋 편집·실행 검증과 구분

| 설정 | 현재 값 |
|---|---|
| Life Cycle Mode | Self (Local 값 1) |
| Loop Behavior | Once (Local 값 1, Infinite는 0) |
| Loop Duration / Delay | 1.2초 / 0초 |
| Initialize Particle Lifetime | 무작위 0.5~0.8초 |
| Particle State / Kill Particles When Lifetime Has Elapsed | true |
| Scale Sprite Size | NormalizedAge 0.3에서 1, 이후 1 유지 |
| Scale Color / Scale Alpha | NormalizedAge 0.15~0.8에서 1, 1.0에서 0 |
| Dynamic Material Parameters / Index 0 Param 1 | NormalizedAge 0~0.5에서 0, 1에서 5 |
| Dynamic Material Parameters / Index 0 Param 2 | NormalizedAge 0~0.5에서 1, 1에서 10 |
| Position Offset | (0,0,120) |

현재 크기 곡선만으로 소멸 설명 불가. 수명 종료·알파 0·나이 기반 머티리얼 변화가 별도 소멸/형태 변화 요소. Param 1/2의 머티리얼 내부 역할은 미확인. 이 Spark2의 설정을 나머지 3개 에미터 전체로 확대하지 않음

Spark2 조정 순서:

1. 복제한 지속형 시스템에서 Emitter Update→Emitter State→Loop Behavior를 Infinite로 변경. Life Cycle Mode=Self·Loop Delay=0 유지. 1.2초 Loop Duration을 0.3초로 임의 변경하지 않음
2. Emitter Update의 Spawn 모듈 확인. Burst만 있으면 Infinite 변경만으로는 다음 루프 사이 공백 가능. 필요한 지속 밀도를 유지하도록 Spawn Rate 사용/조정. 현재 Spawn 설정은 제공되지 않아 Rate·Count 수치 미정
3. Spark2는 개별 불꽃 교체 방식 권장. Lifetime 0.5~0.8초와 Kill Particles=true 우선 유지, Scale Alpha의 개별 페이드 유지. 무조건 Kill을 끄고 지속 스폰을 더하는 방식은 사용하지 않음
4. 후반 머티리얼 형태 변화까지 제거하려면 Dynamic Material Parameters에서 Index 0 Param 1을 Local 0, Param 2를 Local 1로 고정해 비교. 제공 곡선의 전반부 값이며 머티리얼 역할 확인·실제 외형 비교 필요. Param 3의 랜덤과 Param 4=0.05는 유지
5. Scale Sprite Size는 이미 NormalizedAge 0.3 이후 배율 1 유지. 전기 움직임용 SubUVAnimation·Particle Age·Material Time은 계속 진행
6. 탄 중심에서 위로 치우치면 Position Offset Z=120 확인. 원점 부착 의도라면 0으로 조정하되 다른 에미터의 좌표 보정과 함께 검토

입자 자체 나이가 0.3초이면 이 수명 범위에서 NormalizedAge 약 0.375~0.6. 시스템 재생 위치 0.3초에서 모든 입자의 나이가 0.3초인 것은 아님. 크기 곡선의 x=0.3은 수명의 30%이며 해당 입자 생성 후 약 0.15~0.24초. 원본과 동일한 특정 프레임을 정지하는 작업이 아니라 지속 밀도·형태를 유지하는 작업

실제 Spawn 구성·나머지 에미터·머티리얼 내부는 미확인. 위 순서는 Spark2의 지속형 조정안이며 현재 에셋 수정·시각 검증 없음

공식 참고: [Emitter Update](https://dev.epicgames.com/documentation/en-us/unreal-engine/emitter-update-group-reference-for-niagara-effects-in-unreal-engine), [Particle Update](https://dev.epicgames.com/documentation/en-us/unreal-engine/particle-update-group-reference-for-niagara-effects-in-unreal-engine)
