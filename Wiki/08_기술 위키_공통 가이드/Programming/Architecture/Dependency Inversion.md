# Dependency Inversion

## 설계 의도

상위 정책이 하위의 구체 구현에 직접 의존하지 않고 추상화된 경계에 의존하도록 만드는 설계 원칙입니다.

Astra에서는 StateTree Task가 PlayerCharacter를 직접 참조하지 않고 [Entity Context](../../Unreal/StateTree/Entity%20Context.md)를 통해 필요한 Component에 접근하는 방식으로 적용합니다.

---

## 상위 페이지

- [08_기술 위키_공통 가이드](../../08_%EA%B8%B0%EC%88%A0%20%EC%9C%84%ED%82%A4_%EA%B3%B5%ED%86%B5%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)

## 관련

- [시스템 아키텍처](../../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98.md)
- [ADR-001 EntityContext를 참조 경계로 사용](../../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/%EA%B2%B0%EC%A0%95%20%EA%B8%B0%EB%A1%9D/ADR-001%20EntityContext%EB%A5%BC%20%EC%B0%B8%EC%A1%B0%20%EA%B2%BD%EA%B3%84%EB%A1%9C%20%EC%82%AC%EC%9A%A9.md)
