# ADR-001 EntityContext를 참조 경계로 사용

## 설계 의도

## 상태

Accepted

## 문제

StateTree Task가 PlayerCharacter를 직접 참조하면 Task와 특정 Character 구현 사이의 결합도가 높아지고 Player/NPC 공통 사용과 Avatar 교체가 어렵다

## 결정

Task와 Evaluator는 `EntityContext`를 통해 필요한 Component와 런타임 참조를 얻는다

## 이유

- Player/NPC 공통 구조로 확장 가능
- StateTree Task의 Character 의존성 감소
- Possession과 Avatar 교체 대응
- 테스트 용이성 증가

---

## 상위 페이지

- [ADR 목록](ADR%20%EB%AA%A9%EB%A1%9D.md)
- [03_시스템 설계](../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84.md)

## 관련

- [Entity Context](../../08_%EA%B8%B0%EC%88%A0%20%EC%9C%84%ED%82%A4_%EA%B3%B5%ED%86%B5%20%EA%B0%80%EC%9D%B4%EB%93%9C/Unreal/StateTree/Entity%20Context.md)
- [StateTree Evaluator](../../08_%EA%B8%B0%EC%88%A0%20%EC%9C%84%ED%82%A4_%EA%B3%B5%ED%86%B5%20%EA%B0%80%EC%9D%B4%EB%93%9C/Unreal/StateTree/StateTree%20Evaluator.md)
- [Dependency Inversion](../../08_%EA%B8%B0%EC%88%A0%20%EC%9C%84%ED%82%A4_%EA%B3%B5%ED%86%B5%20%EA%B0%80%EC%9D%B4%EB%93%9C/Programming/Architecture/Dependency%20Inversion.md)
- [시스템 아키텍처](../%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98.md)
