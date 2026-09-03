# Prop

## 설계 의도

```mermaid
flowchart LR
    Event["상호작용 · 피해 이벤트"] --> Capability["필요한 Component · 인터페이스"]
    Capability --> State["Prop의 제한된 상태 변경"]
```

## 정의

기획 정의: [Prop](../../02_%EA%B2%8C%EC%9E%84%20%EA%B8%B0%ED%9A%8D/%EA%B0%9D%EC%B2%B4%20%EA%B8%B0%ED%9A%8D/Prop.md)

구현 범위: 클래스·Component·데이터·소유권

## 하위 유형

기획 기준: [Prop](../../02_%EA%B2%8C%EC%9E%84%20%EA%B8%B0%ED%9A%8D/%EA%B0%9D%EC%B2%B4%20%EA%B8%B0%ED%9A%8D/Prop.md)

## Entity와의 경계

기획 기준: [Prop](../../02_%EA%B2%8C%EC%9E%84%20%EA%B8%B0%ED%9A%8D/%EA%B0%9D%EC%B2%B4%20%EA%B8%B0%ED%9A%8D/Prop.md)

## Item과의 경계

기획 기준: [Prop](../../02_%EA%B2%8C%EC%9E%84%20%EA%B8%B0%ED%9A%8D/%EA%B0%9D%EC%B2%B4%20%EA%B8%B0%ED%9A%8D/Prop.md)

## 구현 원칙

- 단순 Prop에 EntityContext, StateTree와 ASC를 일괄 부여하지 않음
- 필요한 Capability만 Component 또는 인터페이스로 조합
- 상호작용 결과는 자신의 제한된 상태만 변경하고 다른 시스템 정책을 직접 소유하지 않음
- 복잡성이 증가해 독립 행동 주체가 되면 이름만 유지하지 말고 Structure Entity로 재분류

---

## 상위 페이지

- [03_시스템 설계](../03_시스템%20설계.md)

## 관련

- [Entity](Entity.md)
- [Item](Item.md)
- [게임 객체 분류](../../02_게임%20기획/게임%20객체%20분류.md)

## 문서 연결

- 기획: [Prop](../../02_%EA%B2%8C%EC%9E%84%20%EA%B8%B0%ED%9A%8D/%EA%B0%9D%EC%B2%B4%20%EA%B8%B0%ED%9A%8D/Prop.md)
- 시스템 설계: [Prop](Prop.md)
- 애니메이션 설계: [Prop](../../04_%EC%95%84%ED%8A%B8%20%EB%B0%8F%20%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98/%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98%20%EC%84%A4%EA%B3%84/Prop.md)
