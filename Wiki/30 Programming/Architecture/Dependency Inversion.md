# Dependency Inversion

상위 정책이 하위의 구체 구현에 직접 의존하지 않고 추상화된 경계에 의존하도록 만드는 설계 원칙입니다.

Astra에서는 StateTree Task가 PlayerCharacter를 직접 참조하지 않고 [[Entity Context]]를 통해 필요한 Component에 접근하는 방식으로 적용합니다.

## 관련

- [[Project Architecture]]
- [[ADR-001 EntityContext를 참조 경계로 사용]]

