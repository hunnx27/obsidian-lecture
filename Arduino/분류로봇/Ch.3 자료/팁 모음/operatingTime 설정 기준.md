---
title: "💡 operatingTime 설정 기준"
parent: "[[팁 모음]]"
source: "Notion"
tags: [arduino, robot, tips, timing]
---

# 💡 operatingTime 설정 기준

## operatingTime이 잘못되면?

```
너무 짧으면:
  모터가 시간 안에 목표 못 도달
  → 중간에서 멈춰 버림
  → 저장값보다 아래 위치에서 정지

너무 길면:
  낭비되지만 안전
  다음 자세로 넘어가는 시간이 늘어남
```

---

## 자세별 권장 operatingTime

| 자세 | 추천 시간 | 이유 |
|------|-----------|------|
| INITIAL | 1000ms | 일반적인 대기 자세 |
| MISSION_1 | 1000ms | 편안한 자세 |
| SEE_STORAGE | 1000ms | 카메라 방향 전환 |
| GRIP_UPPER | **1500ms** | 팔을 멀리 뻗음 |
| GRIP_UPPER_PICK | **1500ms** | 집는 동작 실패 방지 |
| GRIP_LOWER | **1500ms** | 동일 |
| GRIP_LOWER_PICK | **1500ms** | 동일 |
| PUT 자세들 | 1000ms | 내려놓는 동작 |

> ⚠️ GRIP 계열은 팔을 멀리 뻗어서 중력 영향이 큰 자세라 **1500ms 이상 권장**

---

## 테스트 방법

```
SagMeasure.ino에서:

run,6,500    ← 너무 짧음
now          ← 멈춰버린 값 (기대값과 차이 큼)

run,6,1500   ← 충분한 시간
now          ← 저장값에 가까운 값
```

---

## 팔팔 코드에서

```cpp
// 너무 짧은 경우 ❌
RunManipulatorPoseWithPoseDataInEEPROM(dxl, GRIP_UPPER, 500);

// 권장 ✅
RunManipulatorPoseWithPoseDataInEEPROM(dxl, GRIP_UPPER, 1500);
```
