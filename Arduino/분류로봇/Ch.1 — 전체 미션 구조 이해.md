---
title: "🗺️ Ch.1 — 전체 미션 구조 이해"
parent: "[[00 - 색깔 블록 분류 로봇 — 경연 핵심 강의]]"
source: "Notion"
tags: [arduino, robot, lecture]
---

# 🗺️ Ch.1 — 전체 미션 구조 이해

> 코드를 한 줄도 안 봐도 **로봇이 무엇을 하는지** 말할 수 있어야 한다.

---

## 미션 한 줄 요약

**미션지시존에서 블록 순서 파악 → 적재함으로 이동 → 색깔별로 찾아서 집어서 배치**

---

## 전체 흐름 6단계

| 단계  | 동작                      | 핵심 코드                            |
| --- | ----------------------- | -------------------------------- |
| 1   | 초기화 & 버튼 대기             | `InitManipulator`, `OpenGripper` |
| 2   | 장애물 접근 & 미션존 이동         | `PID_AlignAndApproach`           |
| 3   | 블록 전체 스캔 (순서 결정)        | `Pixy_xNUM_OneShot()`            |
| 4   | 적재함 앞 정밀 정렬             | `AlignUsingFrontAndSide430PID`   |
| 5   | 슬롯 순회 → 집기 → 배치 【핵심 루프】 | `Grap_blocks`, `PUT_ON`          |
| 6   | 복귀                      | `Backward`, `INITIAL` 자세         |

---

## 핵심 루프 이해 (가장 중요)

```cpp
while (findBoxNum > 0) {
    // 1. 지금 찾아야 할 색깔 결정
    int targetColor = Pixy_List[count];

    // 2. 슬롯 1→4 순서로 이동하며 탐색
    for (int slot = 0; slot <= NUM_SLOTS; slot++) {
        DriveLeftWithSideLeftSensorPID(...);
        pixy.ccc.getBlocks();

        // 3. 목표 색깔 발견?
        if (blocks[i].m_signature == targetColor) {
            Grap_blocks(y);   // 집기
            PUT_ON(putPos);   // 배치
            count++;
            findBoxNum--;
            break;
        }
    }
}
```

> 💡 **비유:** 4개 서랍을 하나씩 열어보면서, 찾는 색깔 택배가 있으면 꺼내서 지정 칸에 배달하는 것

---

## 🖥️ JSX 시뮬레이션

**파일명:** `robot_flowchart2.jsx`

사용법:
- 각 단계 노드 클릭 → 오른쪽에서 변수 상태 확인
- 파란 점 = 그 단계에서 값이 바뀐 변수
- 루프 반복 과정을 단계별로 확인 가능

---

## ✅ 이해 체크

- [ ] 미션을 한 줄로 설명할 수 있다
- [ ] 핵심 루프에서 count와 findBoxNum이 언제 바뀌는지 안다
- [ ] Pixy_List[]가 왜 필요한지 설명할 수 있다

---

### 관련 자료
- Ch1 시뮬레이션 코드 — `Ch1_MissionStructure.jsx`
