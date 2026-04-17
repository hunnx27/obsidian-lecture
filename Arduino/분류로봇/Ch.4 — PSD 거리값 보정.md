---
title: "📏 Ch.4 — PSD 거리값 보정"
parent: "[[00 - 색깔 블록 분류 로봇 — 경연 핵심 강의]]"
source: "Notion"
tags: [arduino, robot, lecture, psd, sensor]
---

# 📏 Ch.4 — PSD 거리값 보정

> 실습 코드 5개(A~E)로 PSD 제어 패턴을 단계별로 익히고, 경진대회 코드의 목표 ADC값을 직접 실측해서 입력할 수 있다.

---

## 왜 보정이 필요한가

경진대회 원본 코드는 PSD 센서의 **ADC 원시값(0~1023)** 을 목표값으로 사용해.

```cpp
// 원본 경진대회 코드
while(1) {
  GetValueFromSideLeftPSDSensor(&slPSDValue3);
  if (!DriveWithOneSensor(dxl, slPSDValue3 - 518, 5, DRIVE_DIRECTION_LEFT)) break;
}
```

여기서 `518`이 목표 ADC값이야. 이 값은 **특정 환경에서 실측한 값**이라서, 경연장이 다르면 반드시 다시 측정해야 해.

```
경연장 바닥재 다름
  ↓
로봇 높이/무게중심 미세하게 다름
  ↓
PSD 센서가 읽는 ADC값 달라짐
  ↓
목표 ADC값이 맞지 않으면 엉뚱한 위치에 서게 됨
```

---

## PSD 센서 이해

| 변수명 | 핀 | 방향 | 주요 용도 |
|--------|----|----|-----------|
| `PIN_FL` | A0 | 전방 좌측 | 장애물 접근 정지 기준 |
| `PIN_FR` | A2 | 전방 우측 | 2축 정렬 기준 |
| `PIN_SL` | A1 | 좌측면 | 슬롯 이동 기준 ← **핵심!** |
| `PIN_SR` | A3 | 우측면 | 장애물 감지 |

ADC값 특성:
- **가까울수록 → 값 큼** (최대 ~650)
- **멀수록 → 값 작음** (최소 ~100)

---

## 실습 코드 5개 구성

원본 경진대회 코드의 PSD 제어 패턴과 **1:1 대응**:

| 섹션 | 파일명 | 원본 코드 대응 함수 | 역할 |
|------|--------|---------------------|------|
| A | `A_PSD_Monitor.ino` | `GetValueFrom...PSDSensor()` | 4개 센서 실시간 모니터링 |
| B | `B_Front_Approach.ino` | `GoForwardWithTwoSensors()` | FL+FR로 전방 접근 |
| C | `C_Side_Move.ino` | `DriveWithOneSensor()` | SL로 슬롯 위치 이동 |
| D | `D_2Axis_Align.ino` | `LocatingWithTwoSensors()` | SL+FR 2축 직각 정렬 |
| E | `E_Boundary_Return.ino` | `DriveUntilNoObstacleWithOneSensor()` | 장애물 없어질 때까지 이동 |

> ⚠️ **시리얼 설정: 115200 baud / Newline(n)**
> ⚠️ **폴더 구성 규칙**: 각 섹션 폴더에 `Motor.h/cpp`, `Mobilebase.h/cpp`, `Debug.h`, `Pins.h`만 포함. `PSD.cpp`, `Manipulator.cpp`, `Pixy.cpp`, `Gripper.cpp`는 절대 넣지 말 것 → Arduino IDE가 폴더 내 모든 `.cpp`를 자동 컴파일해서 `initDebug()` 중복 정의 에러 발생

---

## 실측 방법 — 섹션A 먼저 실행

### 1단계 — A_PSD_Monitor.ino 업로드

```
FL:312  FR:298  SL:420  SR:185
FL:311  FR:299  SL:421  SR:184
...
SW1 버튼 누르면 종료
```

손을 각 센서 앞에 가져다 대면서 어떤 값이 변하는지 확인.

### 2단계 — 위치별 ADC값 기록

로봇을 각 위치에 수동으로 세우고 값 확인:

| 위치 | 원본 목표 ADC | 내 실측값 | 입력 위치 |
|------|--------------|-----------|-----------|
| 미션지시존 앞 (전방 접근) | FL=530, FR=525 | FL=?, FR=? | `B_Front_Approach.ino` |
| 적재함 2축 정렬 | SL=317, FR=314 | SL=?, FR=? | `D_2Axis_Align.ino` |
| 슬롯1 앞 | SL=420 (예시) | SL=? | `C_Side_Move.ino` |
| 슬롯2 앞 | SL=360 (예시) | SL=? | `C_Side_Move.ino` |
| 슬롯3 앞 | SL=310 (예시) | SL=? | `C_Side_Move.ino` |
| 슬롯4 앞 | SL=270 (예시) | SL=? | `C_Side_Move.ino` |
| 장애물 감지 기준 | FR=220 | FR=? | `E_Boundary_Return.ino` |

### 3단계 — 각 파일 상단 `#define` 수정 후 업로드

```cpp
// B_Front_Approach.ino 상단
#define FL_TARGET  530   // ← 실측값으로 수정
#define FR_TARGET  525   // ← 실측값으로 수정

// C_Side_Move.ino 상단
int16_t SLOT_ADC[4] = {420, 360, 310, 270};  // ← 전부 실측값으로 수정

// D_2Axis_Align.ino 상단
#define SL_TARGET  317   // ← 실측값으로 수정
#define FR_TARGET  314   // ← 실측값으로 수정

// E_Boundary_Return.ino 상단
#define BOUNDARY_ADC  220  // ← 실측값으로 수정
```

---

## 권장 실습 순서

```
1. 섹션A  →  센서 방향 파악 + 각 위치 ADC값 기록
2. 섹션B  →  FL_TARGET, FR_TARGET 입력 후 전방 접근 테스트
3. 섹션D  →  SL_TARGET, FR_TARGET 입력 후 2축 정렬 테스트  ← C보다 먼저!
4. 섹션C  →  SLOT_ADC[] 입력 후 슬롯1→2→3→4 순서 테스트
5. 섹션E  →  BOUNDARY_ADC 입력 후 m으로 모니터링 → r로 실행
```

> 💡 **D를 C보다 먼저 해야 하는 이유**: 적재함 앞에서 직각 정렬이 완료된 상태에서 SL값을 측정해야 슬롯 위치 ADC값이 정확해짐.

---

## 각 섹션 명령어 요약

| 섹션 | 명령어 | 동작 |
|------|--------|------|
| A | (자동) | FL/FR/SL/SR 값 300ms마다 출력, SW1로 종료 |
| B | `1` | 목표 ADC까지 전진 |
| B | `p` | 현재 FL, FR 출력 |
| C | `1`~`4` | 슬롯1~4로 이동 |
| C | `p` | 현재 SL 출력 |
| D | `1` | 2축 정렬 실행 |
| D | `p` | 현재 SL, FR 출력 |
| E | `m` | SL 모니터링 (SW1 종료) |
| E | `r` | 경계 감지 복귀 3단계 실행 |
| E | `p` | 현재 SL 출력 |

---

## 보완 필요한 부분

현재 5개 모듈에 없는 원본 코드 패턴:

**① 카메라 X오차 기반 DriveWithOneSensor** (추가 실습 필요)
원본에서 블록 중앙 정렬 시 카메라 X좌표 오차를 `DriveWithOneSensor`에 입력:

```cpp
int16_t blockXError = pixy.ccc.blocks[0].m_x - PIXY2_X_SETPOINT;
if (!DriveWithOneSensor(dxl, blockXError, PIXY2_X_TOLERANCE, DRIVE_DIRECTION_LEFT)) break;
```

**② 섹션E는 SL 기준이지만 원본은 FR 기준** (핀 변경 테스트 필요)
원본에서 `DriveUntilNoObstacleWithOneSensor`는 FR(A2), 220 기준으로 **우측 이동하며 블록 탐색**. 섹션E에서 `PIN_SL`을 `A2`로 바꿔서 FR 기준 테스트 권장.

**③ 거리 기반 이동 패턴** (별도 모듈 추가 예정)

```cpp
DriveDistanceAndMmPerSecAndDirection(dxl, 450.0);
while(!CheckIfMobilebaseIsInPosition(dxl)) {}
```

PSD 없이 mm 단위로 이동하는 패턴.

---

## ✅ 이해 체크

- [ ] 섹션A로 4개 센서 방향을 손으로 직접 확인할 수 있다
- [ ] 각 위치에서 ADC값을 기록하고 코드에 입력할 수 있다
- [ ] 섹션B~E를 순서대로 실행하고 동작을 확인할 수 있다
- [ ] `DriveWithOneSensor`가 PSD 오차와 카메라 오차 모두에 사용됨을 이해한다
- [ ] `DriveUntilNoObstacleWithOneSensor`의 threshold 의미를 설명할 수 있다
- [ ] ADC 원시값 방식과 cm 방식의 차이를 설명할 수 있다

---

### 관련 자료
- Ch4 시뮬레이션 코드 — `Ch4_PSDCalibration.jsx`
- 🛠️ 모듈 실습 코드 — `Module_Ch4_PSDTest.ino`
- 📊 PSD 노이즈 필터 시뮬레이터 — `Ch4b_PSDNoiseFilter.jsx`
- 📡 PSD 센서 완전 이해 — 종류·측정범위·함수 설명
- 🧪 PSD 모듈 테스트 코드 — `Module_PSD_v2.ino`
- 📡 PSD 5개 모듈 파일 — 실습 가이드
- 📡 PSD 5개 모듈 실습 가이드 (확정판)
- 🔄 ADC → cm 방식 전환 가이드
