# ADR-002 StateTree와 GAS 역할 분리

## 설계 의도

## 상태

Accepted

## 문제

상태 선택과 Ability 실행을 한 시스템에 몰아 넣으면 AI/플레이어 판단, 비용, 쿨다운, 취소 책임이 뒤섞인다

## 결정

StateTree는 행동을 선택하고 GAS는 행동의 비용·쿨다운·실행·취소를 담당

## 이유

- AI와 플레이어가 동일한 Ability 실행 구조를 공유 가능
- 상태 전이와 실행 실패 원인을 분리 가능
- Attribute와 GameplayTag의 소유권이 명확해진다

---

## 상위 페이지

- [ADR 목록](ADR%20%EB%AA%A9%EB%A1%9D.md)
- [03_시스템 설계](../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84.md)

## 관련

- [StateTree와 GAS](../StateTree%EC%99%80%20GAS.md)
- [영혼 결속](../../02_%EA%B2%8C%EC%9E%84%20%EA%B8%B0%ED%9A%8D/%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EA%B8%B0%ED%9A%8D/%EC%98%81%ED%98%BC%20%EA%B2%B0%EC%86%8D.md)
- [시스템 아키텍처](../%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98.md)
