# Project Wiki: 아스트라 — 통합 개발 기준서

이 Wiki는 프로젝트의 현재 구조, 개발 규칙, 검증된 결정과 작업 순서를 관리합니다. 기존 `Project Wiki_Ver2.docx`의 내용을 Unreal Engine 5.8 구현에 맞게 분리·정리했으며, 오래된 Unity 규칙은 [레거시 부록](Appendix/Legacy-Unity-Rules.md)으로 이동했습니다.

## 현재 요약

| 영역 | 현재 상태 | 다음 핵심 작업 |
|---|---|---|
| 프로젝트 초기화 | 완료 | 새 컴포넌트 추가 시 검증 순서 유지 |
| 기본 로코모션 | 완료 | 회귀 테스트 유지 |
| Hanging / Mantle | 완료 | 접촉부 미세 보정은 애니메이션 단계에서 처리 |
| Vault | 기능 구현, 시각 품질 보류 | 전용 애니메이션 확보 후 교체 |
| 무기 장착·수납 | 완료 | 무기별 Anim Layer 확장 |
| 권총 발사 | 기본 경로 완료 | 발사·재장전·조준 GA 확장 |
| 상체 조준 | 진행 중 | Pitch 제한, 총기 정렬, 왼손 IK |
| GAS 전환 | 진행 중 | StatusModel 책임 축소 및 Attribute/Tag 이관 |
| NPC / Soul-Link | 설계 단계 | 플레이어 전투 기반 안정화 후 착수 |

## 핵심 문서

| 문서 | 내용 |
|---|---|
| [01. 아키텍처](01-Architecture.md) | 객체 계층, 데이터 소유권, 초기화 |
| [02. 개발 규칙](02-Development-Rules.md) | Blueprint/C++ 경계, 변경·검증 규칙 |
| [03. StateTree와 GAS](03-StateTree-and-GAS.md) | Base/Upper 분리, Mediator, GAS 역할 |
| [04. 로코모션](04-Locomotion.md) | 이동, Hanging, Mantle, Vault |
| [05. 무기 시스템](05-Weapon-System.md) | DataAsset, Model, Component, Projectile |
| [06. 애니메이션과 조준](06-Animation-and-Aiming.md) | UpperBody, Aim Offset, IK, 카메라 |
| [07. 로드맵](07-Roadmap.md) | 완료 상태와 우선순위 |
| [08. 에디터 절차](08-Editor-Workflows.md) | 반복 작업의 정확한 순서 |

## 최상위 데이터 흐름

```mermaid
flowchart LR
    PC["PlayerController\n입력·조작권·카메라 지시"] --> CH["PlayerCharacter\n엔진 생명주기·요청 진입점"]
    CH --> MED["EntityStateMediatorComponent\n이벤트 허브·상하체 정책"]
    MED --> LOWER["Base StateTree\n전신 로코모션 선택"]
    MED --> UPPER["Upper StateTree\n무기 소지·상체 행동 선택"]
    UPPER --> GAS["Ability System\n행동 실행·비용·쿨다운·차단"]
    LOWER --> LOC["LocomotionComponent\n이동·파쿠르 엔진 API"]
    GAS --> WPN["WeaponComponent\n장착·발사·탄약 중재"]
    PS["AstraPlayerState\nASC·AttributeSet 지속 소유"] --> GAS
    CTX["EntityContext\n참조 가방"] --> LOWER
    CTX --> UPPER
```

## 문서 갱신 규칙

- 기능 완료 시: 해당 Wiki의 상태와 `07-Roadmap.md`를 갱신합니다.
- 일일 종료 시: `Reports/Daily/YYYY/YYYY-MM-DD.md`를 작성합니다.
- 중요한 설계 변경 시: 변경 이유, 대안, 영향을 함께 기록합니다.
- 구현과 문서가 충돌하면 코드를 확인해 현재 사실을 먼저 기록하고, 설계 목표는 별도 항목으로 분리합니다.
