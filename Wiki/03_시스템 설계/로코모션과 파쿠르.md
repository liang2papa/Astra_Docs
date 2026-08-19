# 로코모션과 파쿠르

## 1. 현재 상태

| 기능 | 상태 | 비고 |
|---|---|---|
| Idle / Walk / Sprint | 완료 | Enhanced Input → StateTree 흐름 확인 |
| JumpStart / Jumping / Falling | 완료 | Falling 안정화 |
| Evade | 완료 | 비용·복구 경로 유지 |
| Hanging | 완료 | 정면 판정, 어깨 폭, 재감지 방지 적용 |
| Mantle | 완료 | Root Motion Montage + Motion Warping |
| Vault | 기능 구현 | 시각적으로 부자연스러워 전용 애니메이션 확보 전까지 보류 |

기준 StateTree:

`Content/Astra/Player/StateTree/BaseStateTree/ST_PlayerBaseState.uasset`

## 2. Ledge 감지

### 기본 조건

- 캐릭터가 벽을 바라보고 있어야 합니다.
- 공중 상태에서만 Hanging 후보를 검사합니다.
- 벽의 상단과 전면 Normal을 구합니다.
- 양쪽 어깨 폭을 수용할 수 있는 벽인지 Shape Trace로 검사합니다.
- 캡슐이 들어갈 안전 위치를 확인합니다.

선형 Ray 하나는 점만 검사하므로 어깨 폭이나 올라설 면적을 보장하지 못합니다. 면적 판정에는 Box/Sphere/Capsule Sweep과 여러 샘플을 사용합니다.

## 3. Hanging

- 매달리는 벽과 플레이어 캡슐의 상대 위치는 일정하게 유지합니다.
- 손 높이 기준은 `Ledge Grab ZOffset` 약 55가 현재 기준입니다.
- `HangingWallClearance`는 0을 기준으로 두고 손·무릎 깊이 차이는 애니메이션 또는 IK에서 처리합니다.
- 벽에서 내려가는 동안 캐릭터가 자동으로 뒤돌아 다시 매달리지 않도록 캐릭터 Forward와 벽 방향의 Dot 조건을 사용합니다.

Animation Blueprint의 `LedgeHanging → Fall Loop` 전이가 Inertialization을 사용하면 상태 머신 이후에 Inertialization 노드가 있어야 합니다. 종료 시 경고가 뜨면 전이의 Blend Logic과 AnimGraph의 Inertialization 배치를 함께 확인합니다.

## 4. Mantle

### 판정

Mantle은 다음 두 영역이 모두 확보되어야 합니다.

1. 벽 위의 캡슐 최종 위치가 비어 있음
2. 최종 위치 아래에 실제로 설 수 있는 바닥이 있음

Overlap만으로는 바닥 지지 여부를 알 수 없습니다. 최종 캡슐 위치의 `Overlap/Sweep`과 아래 방향 `Trace`를 분리합니다.

### 실행

- Hanging에서 Jump 또는 W 입력
- Mantle 가능 여부 검사
- 목표 캡슐 위치와 회전 계산
- Motion Warping Target 설정
- Root Motion Montage 재생
- Montage Delegate에서 성공·중단 처리
- 성공 시 최종 캡슐 Transform 보정
- Idle/Walk로 복귀

`Montage_SetEndDelegate`는 해당 AnimInstance와 Montage의 종료 콜백을 교체 등록합니다. 일반 멀티캐스트 Action처럼 호출할 때마다 구독자가 계속 누적되는 형태가 아닙니다. Montage 종료 후 콜백이 호출되며, Component 종료나 중단 경로에서는 활성 참조와 Delegate 의존성을 정리합니다.

### 시각 보정

- 캡슐이 목표 위치에 정확히 도달했는데 손이 뜨거나 파고들면 이동 문제가 아니라 메시 접촉 문제입니다.
- 손은 IK/Control Rig, 무릎은 애니메이션 수정으로 보정합니다.
- Root Motion 후 메시만 아래로 내려갔다 캡슐로 복귀하는 현상은 Montage 구간, Root Motion Mode, Slot/Blend 중복을 점검합니다.

## 5. Vault

프로젝트의 Vault 정의는 **Hanging 상태에서 올라설 공간은 없지만 얇은 담장 너머로 넘어갈 수 있는 경우**입니다.

- 벽이 두껍고 위에 설 공간이 있으면 Mantle입니다.
- Mantle 최종 캡슐 공간 자체가 막혀 있으면 Vault도 불가능합니다.
- 벽 뒤가 낭떠러지여도 Vault 진입을 막지 않습니다. 넘어간 뒤 Falling으로 처리합니다.
- 담장 두께를 완주 거리로 사용하지 않고 현재는 일정한 1/4 호 이동을 사용합니다.

현재 Vault는 전용 애니메이션이 없어 기능만 유지합니다. 동작이 점프와 시각적으로 큰 차이가 없어 신규 애니메이션 또는 Motion Warping용 Vault Montage 확보 후 다시 다듬습니다.

## 6. 회귀 테스트

- [ ] Hanging 진입 시 정면과 어깨 폭 조건 통과
- [ ] 벽을 보지 않으면 Hanging 미진입
- [ ] Hanging에서 뒤로 떨어진 직후 재매달림 없음
- [ ] Mantle 가능 위치에서만 Mantle 진입
- [ ] 머리 위 장애물이 있으면 Mantle/Vault 미진입
- [ ] Mantle 성공 후 Idle/Walk 정상 전환
- [ ] Mantle 후 Jump 시 이전 Ledge에 재매달리지 않음
- [ ] Vault 성공 후 Falling 또는 Ground 상태 정상 복귀
- [ ] 파쿠르 중 피해·사망 시 취소 함수가 MovementMode와 Gravity 복원
