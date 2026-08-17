# ADR-002 StateTree와 GAS 역할 분리

## 상태

Accepted

## 문제

상태 선택과 Ability 실행을 한 시스템에 몰아 넣으면 AI/플레이어 판단, 비용, 쿨다운, 취소 책임이 뒤섞인다.

## 결정

StateTree는 행동을 선택하고 GAS는 행동의 비용·쿨다운·실행·취소를 담당한다.

## 이유

- AI와 플레이어가 동일한 Ability 실행 구조를 공유할 수 있다.
- 상태 전이와 실행 실패 원인을 분리할 수 있다.
- Attribute와 GameplayTag의 소유권이 명확해진다.

## 관련

- [[StateTree and GAS]]
- [[SoulLink Design]]
- [[Project Architecture]]

