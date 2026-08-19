# ADR-001 EntityContext를 참조 경계로 사용

## 상태

Accepted

## 문제

StateTree Task가 PlayerCharacter를 직접 참조하면 Task와 특정 Character 구현 사이의 결합도가 높아지고 Player/NPC 공통 사용과 Avatar 교체가 어렵다.

## 결정

Task와 Evaluator는 `EntityContext`를 통해 필요한 Component와 런타임 참조를 얻는다.

## 이유

- Player/NPC 공통 구조로 확장 가능
- StateTree Task의 Character 의존성 감소
- Possession과 Avatar 교체 대응
- 테스트 용이성 증가

## 관련

- [[Entity Context]]
- [[StateTree Evaluator]]
- [[Dependency Inversion]]
- [[시스템 아키텍처]]

