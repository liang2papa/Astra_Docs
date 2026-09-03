# Slot·AimOffset·IK 적용 순서

```text
기본 무기 Pose → Montage Slot → AimOffset
→ 상·하체 Blend → 필요한 최종 접촉 보정 → Output Pose
```

## 설계 의도

조준 Pose와 발사 애니메이션의 합성 순서 구분. 중복 접촉 보정 방지

## Slot과 Montage

- Skeleton의 Anim Slot Manager에서 Slot 생성
- Montage의 Slot Track과 AnimGraph의 Slot 이름 일치
- Slot은 Montage를 Pose 경로에 합성하는 지점
- Layered Blend Per Bone의 시작 뼈는 Skeleton에 맞춰 지정

## AimOffset 단위와 Base Pose

- 축 범위와 입력 단위 일치
- 도 단위 Pitch 입력을 -1~1 축에 넣으면 작은 회전에도 끝 샘플 도달 가능
- 샘플의 Additive 설정과 Base Pose 통일
- Preview Base Pose와 Sequence 자체의 Additive Base 설정 구분
- 카메라 Pitch 제한과 AimOffset 입력 Clamp는 서로 다른 처리

## 확인된 Astra 사례

- UpperBody Slot이 AimOffset 뒤에 있을 때 Fire Pose가 조준 Pitch를 덮는 현상
- Slot → AimOffset 순서로 해당 경로 수정
- 완성된 애니메이션에 추가한 FABRIK으로 왼손 이탈
- 해당 경로의 중복 FABRIK 제거 후 해결
- 모든 애니메이션에서 IK 제거 또는 같은 합성 순서를 강제하는 규칙은 아님

## 전체 확인·실행 순서

```text
Skeleton Slot 생성
→ Montage Slot Track 지정
→ AnimGraph Slot 연결
→ AimOffset 축·샘플·Base Pose 확인
→ 기본 Pose → Slot → AimOffset → 상하체 Blend
→ 필요한 경우만 최종 Control Rig / IK
→ Output Pose

정상: Idle / Fire / 상하 조준 / 무기 전환 비교
실패: Slot 경로 → Additive 설정 → 중복 IK 순서로 분리 확인
취소: Montage 종료 → 상태 기반 Aim / IK 입력 해제·보간 → 기본 Pose
```

## 근거와 관련 문서

- 대화: 무기 작업 기록 / 해결된 이슈 ANIM-001~003
- [대화·문서·코드 확인 범위](../../%EA%B8%B0%EC%88%A0%20%EB%AC%B8%EC%84%9C%20%EC%A0%95%EB%A6%AC%20%EA%B7%BC%EA%B1%B0.md)
- [Astra 무기 애니메이션 설계](../../../04_%EC%95%84%ED%8A%B8%20%EB%B0%8F%20%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98/%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98%20%EC%84%A4%EA%B3%84/%EB%AC%B4%EA%B8%B0.md)
- [Unreal 지식 목록](../Unreal%20%EC%A7%80%EC%8B%9D%20%EB%AA%A9%EB%A1%9D.md)
- [99_작업 규칙](../../../99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99/99_%EC%9E%91%EC%97%85%20%EA%B7%9C%EC%B9%99.md)
