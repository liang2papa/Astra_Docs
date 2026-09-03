# Soft Reference와 비동기 로딩

```text
TSoftObjectPtr — 어떤 에셋인가
→ FSoftObjectPath — 에셋 경로
→ FStreamableManager.RequestAsyncLoad — 로딩 요청
→ FStreamableHandle — 해당 요청의 추적·관리
→ 완료된 실제 UObject — 화면·컴포넌트에 적용
```

## 설계 의도

아이템 표현 에셋의 로딩 작업 추적. 객체 종료·대상 변경 시 이전 요청 정리

## 역할 구분

- TSoftObjectPtr: 에셋 경로 기반 참조. 참조만으로 로드 완료를 보장하지 않음
- FStreamableManager: 로딩 요청 관리
- FStreamableHandle: 동기·비동기 로딩 작업 핸들
- TSharedPtr<FStreamableHandle>: UObject가 아닌 C++ 핸들의 참조 관리
- 활성 핸들은 로드된 에셋 유지에 관여. [Epic FStreamableHandle](https://dev.epicgames.com/documentation/en-us/unreal-engine/API/Runtime/Engine/FStreamableHandle)

## 선언과 include

```cpp
struct FStreamableHandle;
TSharedPtr<FStreamableHandle> DisplayMeshLoadHandle;
```

멤버 함수를 호출하는 .cpp에는 `Engine/StreamableManager.h` 포함

## 취소·재사용 시 확인

- CancelHandle과 핸들 참조 Reset 구분
- 완료 콜백에 도착한 대상이 현재 요청과 같은지 확인
- 풀 재사용 시 이전 아이템의 Mesh가 새 아이템에 적용되지 않도록 요청 경로·세대 확인
- 취소를 이미 로드된 에셋의 즉시 언로드와 같은 의미로 해석하지 않음

## 개념 실행 순서

```text
표시할 에셋 결정
→ 이전 요청 정리
→ Soft 경로 확인
  ├─ 없음 → 표현 비움 → 종료
  ├─ 이미 로드됨 → 현재 대상에 적용
  └─ 로드 필요 → RequestAsyncLoad → Handle 저장
      ├─ 완료 → 현재 요청·객체 확인 → 실제 Mesh 적용
      ├─ 실패 → 실패 처리, 이전 표현 유지 여부는 정책에 따름
      └─ 종료·대상 변경·풀 반환 → CancelHandle → Handle.Reset
```

## Astra 적용 상태

`FStreamableHandle 설명` 대화의 WorldItem 로딩 예제에서 추출

2026-09-03 현재 `AWorldItemActor` 소스에는 해당 자동 Mesh 로딩·취소 경로 없음

현재 Mesh는 Blueprint 설정. 이 문서는 일반 기술 메모이며 구현 완료 근거 아님


## 근거와 관련 문서

- 대화: [FStreamableHandle 설명](https://chatgpt.com/c/6a93c871-e600-83eb-bc5b-e0d944ddb44d)
- [대화·문서·코드 확인 범위](../../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
- [Soft Texture 아이콘 표시](../UMG/HUD%20%EA%B0%92%EA%B3%BC%20%EC%95%84%EC%9D%B4%EC%BD%98%20%ED%91%9C%EC%8B%9C.md)
- [Astra 적용 상태](../../../03_%EC%8B%9C%EC%8A%A4%ED%85%9C%20%EC%84%A4%EA%B3%84/%EC%9E%A5%EB%B9%84%EC%99%80%20%EC%9D%B8%EB%B2%A4%ED%86%A0%EB%A6%AC.md)
- [Unreal 지식 목록](../Unreal%20%EC%A7%80%EC%8B%9D%20%EB%AA%A9%EB%A1%9D.md)
- [99_작업 규칙](../../../99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99/99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99.md)
