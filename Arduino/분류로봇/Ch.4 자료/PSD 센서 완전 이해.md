---
title: "📡 PSD 센서 완전 이해 — 종류·측정범위·함수 설명"
parent: "[[Ch.4 — PSD 거리값 보정]]"
source: "Notion"
tags: [arduino, robot, psd, sensor, reference]
---

# 📡 PSD 센서 완전 이해 — 종류·측정범위·함수 설명

PSD(Position Sensitive Detector)는 **적외선을 쏘고 반사된 빛의 각도로 거리를 계산**하는 센서야.

> 💡 초음파 센서와 달리 응답 속도가 빠르고 아날로그 전압으로 거리를 출력해서 Arduino의 `analogRead()`로 바로 읽을 수 있어.

---

## 📦 이 로봇에 달린 PSD 센서 4개

| 핀 이름 | 핀 번호 | 위치 | 센서 모델 추정 | 측정 범위 |
|---------|---------|------|----------------|-----------|
| `PIN_FRONT_LEFT_PSD` | A0 | 전방 좌측 | GP2Y0A21YK0F | 10 ~ 80cm |
| `PIN_SIDE_LEFT_PSD` | A1 | 좌측면 | GP2Y0A21YK0F | 10 ~ 80cm |
| `PIN_FRONT_RIGHT_PSD` | A2 | 전방 우측 | GP2Y0A21YK0F | 10 ~ 80cm |
| `PIN_SIDE_LEFT430_PSD` | A3 | 좌측면(근거리) | GP2Y0A41SK0F | **4 ~ 30cm** |

> ⚠️ `430` = **4cm~30cm** 의미. SP=6cm처럼 가까운 거리는 430 센서로 측정!

---

## ⚠️ 측정 범위 벗어나면?

```
10cm 미만 (GP2Y0A21 기준):
  → 값이 튀거나 최대값 출력
  → PID 오작동 → 벽 충돌 위험

80cm 초과:
  → 0 또는 포화값 출력
  → PID가 계속 이동 → 타임아웃으로 강제 정지
```

```
SLOT_PSD 거리 범위 확인:
  17.5cm ✅  24.5cm ✅  31.0cm ✅  35.5cm ✅  (모두 10~80cm 범위 내)
  SP = 6cm ⚠️ → 430 센서 사용 이유!
```

---

## 🔧 PSD 관련 함수 전체 설명

### 1. `InitPSD()`

```cpp
void InitPSD() {
  // readings[][] 배열을 0으로 초기화
  // total[], average[] 배열을 0으로 초기화
  // 모든 PSD 채널 초기 안정화
}
```

**하는 일:** 이동평균 필터에 쓰이는 배열들을 전부 0으로 초기화. `setup()`에서 반드시 호출해야 해. 안 하면 쓰레기값이 들어있어서 처음 몇 번 측정값이 튈 수 있어.

---

### 2. `FL_PSD_Save()` / `FR_PSD_Save()` — 전방 PSD

```cpp
float FL_PSD_Save() {
  // 1. analogRead(PIN_FRONT_LEFT_PSD) 로 새 값 읽기
  // 2. 이동평균 필터 적용 (최근 NUM_READINGS개 평균)
  // 3. cm 단위로 변환해서 average[FL]에 저장
  return average[FL];
}
```

**하는 일:** 전방 왼쪽/오른쪽 거리를 cm로 반환. 주로 `PID_Approach`, `AlignUsingFrontAndSide430PID`에서 앞쪽 장애물까지 거리 측정에 사용.

```cpp
// 전방 PSD로 40cm까지 접근
PID_Approach(dxl, 40, 10, 0.01, 0.2); // 내부적으로 FL_PSD_Save() 호출
```

---

### 3. `SL_PSD_Save()` — 좌측면 PSD (슬롯 이동 기준!)

```cpp
float SL_PSD_Save() {
  // analogRead(PIN_SIDE_LEFT_PSD) → 이동평균 → cm 변환
  return average[SL];
}
```

**하는 일:** 로봇 왼쪽 옆면에서 왼쪽 벽까지 거리 반환. **슬롯 이동의 핵심 기준값.**

```cpp
// 슬롯 이동 시 내부에서 계속 호출됨
DriveLeftWithSideLeftSensorPID(dxl, SLOT_PSD[0], ...);
// 함수 내부: while (SL_PSD_Save() != target) { 이동 }

// 복귀 시 벽 감지에도 사용
while (1) {
  float dis = SL_PSD_Save();
  Backward(dxl, 250);
  if (dis > 15) break; // 벽이 안 보이면 탈출
}
```

---

### 4. `SL430_PSD_Save()` — 좌측면 근거리 PSD

```cpp
float SL430_PSD_Save() {
  // analogRead(PIN_SIDE_LEFT430_PSD) 읽기
  // GP2Y0A41 (4~30cm) 전용 변환 공식 적용
  return average[SL430];
}
```

**하는 일:** 4~30cm 근거리 전용. `AlignUsingFrontAndSide430PID`에서 정렬 시 옆면 기준으로 사용. SP=6cm처럼 일반 SL이 측정 못 하는 가까운 거리를 담당.

```cpp
// 8단계 정렬: 앞 43cm + 옆 430 6cm 기준으로 직각 정렬
AlignUsingFrontAndSide430PID(dxl, FP=43, SP=6,
                             0.5, 0.5,
                             5, 0.01, 0.1,
                             2, 0.01, 0.03,
                             50, 150);
// 앞: FL_PSD_Save() 기준 43cm
// 옆: SL430_PSD_Save() 기준 6cm
```

---

### 5. `PrintPSDvalue()`

```cpp
void PrintPSDvalue() {
  Serial.print("FL: "); Serial.print(average[FL]);
  Serial.print(" FR: "); Serial.print(average[FR]);
  Serial.print(" SL: "); Serial.print(average[SL]);
  Serial.print(" SL430: "); Serial.println(average[SL430]);
}
```

**하는 일:** 4개 센서 현재값을 시리얼로 출력. 디버깅·실측 시 필수.

---

### 6. `DriveLeftWithSideLeftSensorPID()` — 슬롯 이동 핵심 함수

```cpp
void DriveLeftWithSideLeftSensorPID(
  Dynamixel2Arduino dxl,
  float targetDist,   // 목표 거리 (cm) — 옆벽 기준!
  float minSpeed,     // 최소 속도 (0.2 권장)
  float maxSpeed,     // 최대 속도 (1.0 권장)
  float ki,           // 적분 게인
  float kd,           // 미분 게인
  int stopThreshold,  // 정지 허용 오차 (mm)
  int timeout         // 타임아웃 (ms)
) {
  while (1) {
    float current = SL_PSD_Save(); // 왼쪽 옆벽 거리 읽기
    float error = targetDist - current;

    if (abs(error) < stopThreshold) break; // 목표 도달!

    float speed = PID계산(error, ki, kd);
    speed = constrain(speed, minSpeed, maxSpeed);

    if (error > 0) DriveLeft(speed);
    else           DriveRight(speed);
  }
  Stop();
}
```

**핵심 포인트:**
- `targetDist`는 왼쪽 **옆벽**까지의 거리
- 적재함이 아니라 경연장 옆벽이 기준
- PID가 overshooting 없이 부드럽게 정지

---

### 7. `AlignUsingFrontAndSide430PID()` — 2축 정밀 정렬

```cpp
void AlignUsingFrontAndSide430PID(
  Dynamixel2Arduino dxl,
  float frontTarget,  // 앞 목표 거리 FP=43cm
  float sideTarget,   // 옆 목표 거리 SP=6cm
  float frontWeight,  // 앞 센서 가중치 0.5
  float sideWeight,   // 옆 센서 가중치 0.5
  // ... PID 파라미터들
);
```

**하는 일:** 앞(FL_PSD)과 옆(SL430_PSD)을 동시에 맞춰서 로봇이 적재함에 **직각**으로 정렬되게 함. 이 정렬이 완료된 상태에서 슬롯 이동을 해야 SL PSD값이 정확해.

```
정렬 전:           정렬 후:
   /  ← 로봇         │  ← 로봇
[적재함]          [적재함]

SL값 부정확       SL값 정확 → 슬롯 이동 가능!
```

---

## 📋 PSD 사용 흐름 요약

```
setup()
  └─ InitPSD()  ← 배열 초기화

loop() / 각 단계
  ├─ FL_PSD_Save()    ← 전방 거리 (PID_Approach용)
  ├─ FR_PSD_Save()    ← 전방 거리 (정렬용)
  ├─ SL_PSD_Save()    ← 옆면 거리 (슬롯 이동 기준)
  ├─ SL430_PSD_Save() ← 근거리 옆면 (정밀 정렬용)
  └─ PrintPSDvalue()  ← 디버깅 출력

함수별 사용 센서:
  DriveLeftWithSideLeftSensorPID → SL_PSD
  AlignUsingFrontAndSide430PID   → FL_PSD + SL430_PSD
  PID_Approach                   → FL_PSD
  복귀 경계 감지                  → SL_PSD
```
