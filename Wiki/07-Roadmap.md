# 로드맵과 완료 기준

## 1. 단계별 현황

| 단계 | 목표 | 완료 조건 | 상태 |
|---|---|---|---|
| P0 | 프로젝트 연결 | `.uproject` 직접 실행, UE 5.8 Editor 빌드 | 완료 |
| P1 | 초기화 안정화 | 중복 호출·재소유에도 Context, StateTree, Input 정상 | 완료 |
| P2 | 상태 정합성 | 상태별 단일 의미와 Enter/Exit 복구 경로 | 완료 |
| P3 | 기본 TPS 이동 | Idle/Walk/Sprint/Jump/Fall 안정 | 완료 |
| P4 | 공중·회피 | Evade 비용, Jump/Fall, 중단 복구 | 완료 |
| P5 | 파쿠르 | Hanging/Mantle/Vault 성공·중단 복구 | 기능 완료, Vault 시각 보류 |
| P6 | 무기 기반 | 권총 장착·수납·발사·Upper StateTree | 진행 중 |
| P7 | 조준·재장전 | Aim/Reload GA, 카메라와 손 정렬 | 진행 중 |
| P8 | 장비·인벤토리 | WeaponModel 보존, 예비 탄약, 교체 UI | 대기 |
| P9 | NPC 전투 | NPC ASC/AbilitySet, 성향 기반 명중률 | 대기 |
| P10 | Soul-Link | Possession/DirectSkill/Transform/Totem | 설계 |

## 2. 완료된 핵심 작업

### 로코모션

- Walk, Sprint, Jump, Falling, Evade
- 어깨 폭과 정면 조건을 포함한 Hanging
- Root Motion + Motion Warping Mantle
- Mantle 후 Falling/재매달림/중력 복구
- Vault 상태와 이동 취소 함수

### 무기

- 권총 Static Mesh 장착
- WeaponDataAsset / WeaponModel / WeaponComponent 책임 분리
- 손/홀스터/Muzzle Socket
- Draw/Holster Montage와 Notify
- Upper StateTree에 Carry와 Action 통합
- 자동 Holster Timer
- Blueprint 기반 `GA_AstraBase → GA_WeaponBase → GA_WeaponFire`
- 카메라 Ray 목표와 Muzzle 기반 Projectile 방향 수정
- Fire/Aim 진입 시 Sprint 취소 정책

### 아키텍처

- AstraPlayerState에 ASC와 AttributeSet 배치
- EntityStateMediatorComponent 이벤트 허브
- Base/Upper StateTree 간 GameplayTag 이벤트 중재
- EntityContext에 ASC, Health, Mediator, EventBus 참조 추가

## 3. 현재 최우선 작업

1. 카메라 Pitch를 ±70도로 제한
2. Aim Offset 입력·보간 회귀 테스트
3. 총기와 오른팔이 AimTarget을 향하도록 시각 보정
4. `LeftHandGrip` Socket과 왼손 IK
5. `GA_WeaponReload` 및 Reload Montage/Notify
6. `GA_WeaponAim`의 Hold/Toggle 옵션과 카메라 Zoom
7. Fire Montage, 반동, Muzzle VFX, Sound Concurrency

## 4. 미결정·보류 항목

- Vault 전용 애니메이션과 최종 이동 방식
- 무기별 Anim Layer 구조의 확장 시점
- 표면 재질별 Projectile Impact VFX/Sound
- NPC ASC의 정확한 소유 클래스
- 인벤토리 도입 후 WeaponModel 소유권 이전
- Soul-Link AbilitySet 교체 방식과 NPC AI/Player Control 공존 규칙

## 5. 완료 정의

기능은 다음을 모두 만족해야 완료로 기록합니다.

- 정상 경로가 동작함
- 실패·중단·사망 경로에서 상태가 복구됨
- StateTree, GAS, Component 중 단일 소유권이 명확함
- Blueprint 런타임 None 오류가 없음
- 에디터 종료 시 AnimGraph/Delegate 경고가 없음
- 로그와 플레이 테스트 결과가 일일 리포트에 기록됨
