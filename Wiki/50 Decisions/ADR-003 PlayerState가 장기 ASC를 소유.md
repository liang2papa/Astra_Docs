# ADR-003 PlayerState가 장기 ASC를 소유

## 상태

Accepted

## 문제

영혼 결속의 빙의·변신으로 Pawn이 교체될 때 ASC와 쿨다운을 Pawn이 소유하면 장기 상태가 사라질 수 있다.

## 결정

플레이어의 장기 ASC와 AttributeSet은 PlayerState가 소유하고 현재 Pawn을 Avatar로 사용한다.

## 이유

- Pawn 교체 후에도 자원과 쿨다운 유지
- Possession 구조와 자연스럽게 결합
- 입력 주체(PlayerController), 장기 상태(PlayerState), 표현(Pawn)의 책임 분리

## 관련

- [[Project Architecture]]
- [[SoulLink Design]]
- [[StateTree and GAS]]

