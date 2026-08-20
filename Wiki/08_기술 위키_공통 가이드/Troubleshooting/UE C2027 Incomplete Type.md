# UE - C2027 Incomplete Type

## 증상

Forward Declaration만 있는 Unreal 타입을 실제 정의가 필요한 위치에서 사용하면 C2027이 발생한다.

## 원인

컴파일러가 해당 타입의 전체 정의를 보지 못했다.

## 해결

실제 타입 정의가 필요한 파일에서 해당 Unreal 헤더를 include한다.

---

## 상위 페이지

- [[Troubleshooting 목록]]
- [[08_기술 위키_공통 가이드]]

## 관련

- [[Unreal 지식 목록]]

