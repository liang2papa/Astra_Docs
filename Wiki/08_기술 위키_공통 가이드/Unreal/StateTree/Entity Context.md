# Entity Context

## 정의

StateTree Task와 Evaluator가 구체 Character를 직접 참조하지 않도록 필요한 Component와 런타임 참조를 제공하는 Context 계층입니다.

## 장점

- Player/NPC 공통 Task로 확장하기 쉽다.
- 구체 Character 의존성을 줄인다.
- 테스트와 Avatar 교체에 유리하다.

## 프로젝트 적용

- [[시스템 아키텍처]]
- [[ADR-001 EntityContext를 참조 경계로 사용]]

---

## 상위 페이지

- [[Unreal 지식 목록]]
- [[08_기술 위키_공통 가이드]]

## 관련

- [[StateTree Evaluator]]
- [[Dependency Inversion]]

