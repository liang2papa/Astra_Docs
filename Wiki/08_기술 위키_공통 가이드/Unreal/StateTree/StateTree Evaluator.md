# StateTree Evaluator

```text
외부 Actor / Context → Evaluator 데이터 제공
→ Condition 판단 → 전이 선택 → Task 실행
```

## 설계 의도

StateTree 노드의 구체 Character 의존 감소. 필요한 참조의 공통 준비

## 역할

| 항목 | 책임 |
|---|---|
| Evaluator | 상황·참조·계산 결과를 바인딩 데이터로 제공 |
| Condition | 전이·상태 선택 조건 판단 |
| Task | 상태 실행·종료 처리 |
| Transition | 조건·이벤트에 따른 상태 이동 |

## 실행 시점

- Tree Start: 참조 확보·초기 데이터 준비
- Tick: 계속 변하는 관찰값 갱신이 필요한 경우
- Tree Stop: 자신이 등록한 관찰·구독 정리
- 시작·종료·Tick 지원: [Epic Evaluator API](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Plugins/StateTreeModule/UStateTreeEvaluatorBlueprintBase)

객체 참조를 시작 시 한 번 준비하는 용도로도 사용 가능

Pawn·Context가 교체되는 구조에서는 이전 참조 무효화와 새 참조 갱신 필요

## Context와의 차이

- Context: 필요한 외부 객체를 전달하는 경계
- Evaluator: 실행 중 사용할 데이터를 준비하는 노드
- Astra EntityContext: 프로젝트가 정의한 참조 가방. 엔진 Context Data 전체와 같은 개념은 아님
- Evaluator 도입과 Actor + EntityContext 직접 바인딩 중 실제 구성은 프로젝트 설계에서 결정
- 과거 대화의 Evaluator_PlayerContext 예시를 현재 C++ 클래스 존재로 해석하지 않음

## 개념 실행 순서

```text
StateTree 시작 → Tree Start → Context / 객체 참조 확보
→ Condition에 데이터 바인딩 → 조건 판단 → Task 실행
→ 필요한 관찰값만 Tick 갱신
→ StateTree 종료 → Tree Stop → 소유한 구독·캐시 정리

참조 획득 실패 → 실패를 드러내고 해당 노드 실행 경로 확인
Pawn 교체 → 이전 참조 정리 → 새 Context로 다시 초기화
```

## 근거와 관련 문서

- 대화: [StateTree Evaluator 역할](https://chatgpt.com/c/6a9283f9-2a00-83ed-83c7-1adc7b60c247)
- [대화·문서·코드 확인 범위](../../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
- [Entity Context](Entity%20Context.md)
- [Astra StateTree와 GAS](../../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/StateTree%EC%99%80%20GAS.md)
- [Unreal 지식 목록](../Unreal%20%EC%A7%80%EC%8B%9D%20%EB%AA%A9%EB%A1%9D.md)
- [99_작업 규칙](../../../99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99/99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99.md)
