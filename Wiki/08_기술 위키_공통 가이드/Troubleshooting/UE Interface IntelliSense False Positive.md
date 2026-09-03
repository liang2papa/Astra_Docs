# UE - Interface IntelliSense False Positive

## 설계 의도

## 증상

`IAbilitySystemInterface` 관련 오류 표시가 IDE에 나타난다.

## 원인

IntelliSense 분석과 Unreal Build Tool의 실제 빌드 결과가 다를 수 있다.

## 해결

IDE 표시만으로 판단하지 않고 Unreal Development Editor 전체 빌드로 검증한다. 2026-08-12 기록에서는 실제 빌드가 성공했다.

---

## 상위 페이지

- [Troubleshooting 목록](Troubleshooting%20%EB%AA%A9%EB%A1%9D.md)
- [08_기술 위키_공통 가이드](../08_%EA%B8%B0%EC%88%A0%20%EC%9C%84%ED%82%A4_%EA%B3%B5%ED%86%B5%20%EA%B0%80%EC%9D%B4%EB%93%9C.md)

## 관련

- [StateTree와 GAS](../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/StateTree%EC%99%80%20GAS.md)
- [Editor 작업 절차](../Editor%20%EC%9E%91%EC%97%85%20%EC%A0%88%EC%B0%A8.md)
