# UE - Interface IntelliSense False Positive

## 증상

`IAbilitySystemInterface` 관련 오류 표시가 IDE에 나타난다.

## 원인

IntelliSense 분석과 Unreal Build Tool의 실제 빌드 결과가 다를 수 있다.

## 해결

IDE 표시만으로 판단하지 않고 Unreal Development Editor 전체 빌드로 검증한다. 2026-08-12 기록에서는 실제 빌드가 성공했다.

## 관련

- [[StateTree and GAS]]
- [[08-Editor-Workflows]]

