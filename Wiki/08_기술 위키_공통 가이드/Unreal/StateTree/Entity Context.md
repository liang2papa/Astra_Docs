# Entity Context

## 정의

StateTree Task와 Evaluator가 구체 Character를 직접 참조하지 않도록 필요한 Component와 런타임 참조를 제공하는 Context 계층입니다.

## 장점

- Player/NPC 공통 Task로 확장하기 쉽다.
- 구체 Character 의존성을 줄인다.
- 테스트와 Avatar 교체에 유리하다.

## 프로젝트 적용

- [시스템 아키텍처](../../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98.md)
- [ADR-001 EntityContext를 참조 경계로 사용](../../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/%EA%B2%B0%EC%A0%95%20%EA%B8%B0%EB%A1%9D/ADR-001%20EntityContext%EB%A5%BC%20%EC%B0%B8%EC%A1%B0%20%EA%B2%BD%EA%B3%84%EB%A1%9C%20%EC%82%AC%EC%9A%A9.md)

---

## 상위 페이지

- [Unreal 지식 목록](../Unreal%20%EC%A7%80%EC%8B%9D%20%EB%AA%A9%EB%A1%9D.md)
- [08_기술 위키_공통 가이드](../../08_%EA%B8%B0%EC%88%A0%20%EC%9C%84%ED%82%A4_%EA%B3%B5%ED%86%B5%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)

## 관련

- [StateTree Evaluator](StateTree%20Evaluator.md)
- [Dependency Inversion](../../Programming/Architecture/Dependency%20Inversion.md)

