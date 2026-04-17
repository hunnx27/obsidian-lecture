---
title: "📡 PSD 5개 모듈 실습 가이드 (확정판)"
parent: "[[Ch.4 — PSD 거리값 보정]]"
source: "Notion"
tags: [arduino, robot, psd, guide, final]
---

# 📡 PSD 5개 모듈 실습 가이드 (확정판)

## 개요

이 5개 파일은 **원본 경진대회 코드(Blackberry-15-01)의 PSD 제어 패턴을 1:1로 분리**해서 단독 테스트할 수 있도록 만든 실습 코드야.

> ⚠️ **시리얼 설정: 115200 baud / Newline(n)**

---

## 폴더 구성 규칙 (공통)

각 섹션 폴더에 넣을 파일:

```
섹션X_폴더/
├── X_파일명.ino       ← 이 파일만
├── Motor.h
├── Motor.cpp
├── Mobilebase.h
├── Mobilebase.cpp
├── Debug.h            ← .h만 (cpp 없음)
└── Pins.h             ← .h만 (cpp 없음)
```

❌ **절대 넣으면 안 되는 파일** (`initDebug()` 중복 정의 에러 발생)
- `PSD.cpp` / `PSD.h`
- `Manipulator.cpp` / `Manipulator.h`
- `Pixy.cpp` / `Pixy.h`
- `Gripper.cpp` / `Gripper.h`
- `RGBLED.h`

> 💡 Arduino IDE는 폴더 안 모든 .cpp를 `#include` 없이도 자동 컴파일함. 그래서 불필요한 .cpp는 물리적으로 제거해야 함.

---

## 섹션 A — 실시간 PSD 값 읽기

**파일:** `A_PSD_Monitor.ino`
**원본 코드 대응:**

```cpp
GetValueFromFrontPSDSensors(&flPSDValue2, &frPSDValue2);
GetValueFromSideLeftPSDSensor(&slPSDValue3);
```

**학습 목표:** 4개 센서 방향 파악 + 목표 ADC값 실측

**동작:** 업로드하면 자동으로 FL/FR/SL/SR 값을 300ms마다 출력. SW1 버튼 누르면 종료.

> 💡 **이 섹션에서 확인한 값을 B, C, D, E의 목표값으로 입력해야 함!**

---

## 섹션 B — 전방 PSD 접근

**파일:** `B_Front_Approach.ino`
**원본 코드 대응:**

```cpp
// 전방으로 이동하기
while(1) {
  GetValueFromFrontPSDSensors(&flPSDValue2, &frPSDValue2);
  if (!GoForwardWithTwoSensors(dxl, flPSDValue2 - 530, frPSDValue2 - 525, 5)) break;
}
```

**학습 포인트:**
- `오차 = 현재ADC - 목표ADC` → 0이 되면 정지
- FL=530, FR=525가 장애물 앞 특정 거리에서의 ADC값
- 두 센서를 동시에 맞추므로 좌우 기울기도 보정됨

**명령어:**

| 명령어 | 동작 |
|--------|------|
| `1` | FL_TARGET, FR_TARGET까지 전진 |
| `p` | 현재 FL, FR ADC값 출력 |
| `?` | 가이드 출력 |

---

## 섹션 C — 측면 PSD 좌우 이동

**파일:** `C_Side_Move.ino`
**원본 코드 대응:**

```cpp
// 좌측으로 이동하기
while(1) {
  GetValueFromSideLeftPSDSensor(&slPSDValue3);
  if (!DriveWithOneSensor(dxl, slPSDValue3 - 518, 5, DRIVE_DIRECTION_LEFT)) break;
}

// 블록 X좌표 중앙 정렬 (카메라 오차 기반)
if (!DriveWithOneSensor(dxl, blockXError, PIXY2_X_TOLERANCE, DRIVE_DIRECTION_LEFT)) break;

// 미션수행존으로 좌측 이동
while(1) {
  GetValueFromSideLeftPSDSensor(&slPSDValue9);
  if (!DriveWithOneSensor(dxl, slPSDValue9 - 595, 5, DRIVE_DIRECTION_LEFT)) break;
}
```

**학습 포인트:**
- SL=518 → 미션지시존 앞 위치
- SL=595 → 미션수행존 앞 위치
- `DriveWithOneSensor`는 PSD 오차 외에 카메라 X오차도 처리 가능 (같은 함수)

**명령어:**

| 명령어 | 동작 |
|--------|------|
| `1~4` | 슬롯1~4로 이동 |
| `p` | 현재 SL ADC값 출력 |
| `?` | 가이드 출력 |

---

## 섹션 D — 2축 동시 정밀 정렬

**파일:** `D_2Axis_Align.ino`
**원본 코드 대응:**

```cpp
// 좌측, 전방 우측 PSD 값 맞춰서 이동하기
while(1) {
  GetValueFromSideLeftPSDSensor(&slPSDValue6);
  GetValueFromFrontRightPSDSensor(&frPSDValue6);
  if (!LocatingWithTwoSensors(dxl, slPSDValue6 - 317, frPSDValue6 - 314, 5, DRIVE_DIRECTION_LEFT)) break;
}
```

**학습 포인트:**
- SL=317, FR=314 → 적재함 앞 직각 정렬 완료 위치
- 두 센서 오차를 동시에 0으로 만들어 직각 맞춤
- **이 정렬 후 섹션C 이동을 해야 슬롯 위치가 정확해짐**

**명령어:**

| 명령어 | 동작 |
|--------|------|
| `1` | SL_TARGET + FR_TARGET 기준 2축 정렬 |
| `p` | 현재 SL, FR ADC값 출력 |
| `?` | 가이드 출력 |

---

## 섹션 E — 장애물 감지 이동

**파일:** `E_Boundary_Return.ino`
**원본 코드 대응:**

```cpp
// FR PSD에 장애물 없어질 때까지 우측으로 이동
do {
  GetValueFromFrontRightPSDSensor(&frPSDValue7);
} while (DriveUntilNoObstacleWithOneSensor(dxl, frPSDValue7, 220, DRIVE_DIRECTION_RIGHT));

// SL PSD 값 맞춰서 좌측으로 이동 (2번째 블록 탐색)
do {
  GetValueFromSideLeftPSDSensor(&slPSDValue8);
} while (DriveWithOneSensor(dxl, slPSDValue8 - 526, 5, DRIVE_DIRECTION_LEFT));
```

**학습 포인트:**
- `ADC < threshold` → 장애물 없음으로 판단, 정지
- FR 기준 220 → 이 값 이하가 되면 벽/장애물이 없다고 판단
- **실습 코드는 SL 기준으로 구현돼 있음 (FR 기준도 직접 바꿔서 테스트 권장)**

**명령어:**

| 명령어 | 동작 |
|--------|------|
| `m` | SL 모니터링 (SW1=종료) |
| `r` | 경계 감지 복귀 3단계 실행 |
| `p` | 현재 SL ADC값 출력 |
| `?` | 가이드 출력 |

---

## 교육 흐름 평가

### ✅ 잘 된 부분

원본 코드의 PSD 제어 패턴 5가지가 섹션 A~E에 **1:1로 대응**됨.

```
원본 코드                           →  실습 섹션
GoForwardWithTwoSensors              →  섹션 B
DriveWithOneSensor (SL 기준)         →  섹션 C
LocatingWithTwoSensors               →  섹션 D
DriveUntilNoObstacleWithOneSensor    →  섹션 E
GetValueFrom...PSDSensor (값 읽기)   →  섹션 A
```

### ⚠️ 보완할 부분

**1. 카메라 X오차 기반 DriveWithOneSensor 미포함**

```cpp
// 원본 코드
int16_t blockXError = pixy.ccc.blocks[0].m_x - PIXY2_X_SETPOINT;
if (!DriveWithOneSensor(dxl, blockXError, PIXY2_X_TOLERANCE, DRIVE_DIRECTION_LEFT)) break;
```

섹션C는 PSD 오차만 연습하므로, 카메라+PSD 혼합 제어는 별도 실습 필요.

**2. 섹션E가 FR 기준이 아닌 SL 기준으로 구현됨**

원본에서 `DriveUntilNoObstacleWithOneSensor`는 **FR(전방우측)** 기준이지만, 실습 코드는 SL 기준. 실습 시 FR 핀으로 바꿔서도 테스트 권장.

```cpp
// 수정 위치: E_Boundary_Return.ino
#define PIN_SL  A1   // ← 이걸
#define PIN_SL  A2   // ← FR(A2)로 바꿔서 테스트
#define BOUNDARY_ADC  220  // FR 기준값 유지
```

**3. 거리 기반 이동 패턴 미포함**

```cpp
DriveDistanceAndMmPerSecAndDirection(dxl, 450.0);
while(!CheckIfMobilebaseIsInPosition(dxl)) {}

DriveXYDistanceAndMmPerSec(dxl, 60.0, 350.0);
while(!CheckIfMobilebaseIsInPosition(dxl)) {}
```

---

## 수정본(v2)으로 업그레이드 가능한지

**결론: 가능하지만 함수를 새로 만들어야 함.**

| 항목 | 원본 (이번 실습) | 수정본 |
|------|------------------|--------|
| 거리 단위 | ADC 원시값 (0~1023) | cm |
| 제어 방식 | 단순 오차 → threshold | PID 게인 |
| 목표값 설정 | 현장 ADC 실측 필수 | cm 직접 입력 |
| 함수 | `GoForwardWithTwoSensors` 등 | `PID_AlignAndApproach` 등 |

수정본의 `PID_AlignAndApproach`, `DriveLeftWithSideLeftSensorPID` 등은 원본 `Mobilebase.h`에 없는 별도 구현 함수야. 원본을 수정본 방식으로 바꾸려면 이 함수들을 `Mobilebase.cpp`에 추가 구현해야 해.

---

## 권장 실습 순서

```
1. 섹션A → 각 센서 방향 파악 + 위치별 ADC값 기록
2. 섹션B → 전방 접근 목표값 입력 후 테스트
3. 섹션D → 적재함 앞 2축 정렬 목표값 입력 후 테스트
4. 섹션C → 슬롯별 SL값 입력 후 1→2→3→4→기준점 순서 테스트
5. 섹션E → BOUNDARY_ADC 입력 후 m으로 검증 → r로 실행
```
