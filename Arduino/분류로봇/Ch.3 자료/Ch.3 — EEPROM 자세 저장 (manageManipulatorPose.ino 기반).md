---
title: "🦾 Ch.3 — EEPROM 자세 저장 (manageManipulatorPose.ino 기반)"
parent: "[[Ch.3 — EEPROM 자세 저장]]"
source: "Notion"
tags: [arduino, robot, eeprom, ino]
---

# 🦾 Ch.3 — EEPROM 자세 저장 (manageManipulatorPose.ino 기반)

## 🎯 이 챕터의 목표

> 로봇팔을 손으로 원하는 위치로 이동하고, 그 각도를 EEPROM에 저장한 뒤, 다시 불러와서 실행까지 확인한다.

---

## 📦 필요한 파일 구성

아래 파일들이 **같은 폴더**에 있어야 해.

| 파일 | 역할 |
|------|------|
| `manageManipulatorPose.ino` | 메인 실행 파일 |
| `Debug.h` | 시리얼 통신 설정 (`DEBUG_SERIAL`, `SERIAL_BAUD_RATE` 정의) |
| `Motor.h` | 다이나믹셀 통신 초기화 (`initMotorCommunication()`) |
| `Manipulator.h` | EEPROM 저장/불러오기 함수들 |
| `RGBLED.h` | 성공(초록)/실패(빨강) LED 피드백 |
| `Pins.h` | RGB LED 핀 정의 (`RGB_LED_PIN`) |

> ⚠️ **중요:** 처음 사용하는 보드라면 반드시 Arduino 예제의 `EEPROM > eeprom_clear`를 먼저 한 번 실행해서 쓰레기 값을 초기화해야 해.

---

## 🔌 업로드 전 준비

### 1단계 — 활성화해야 할 줄 확인

현재 코드에서 아래 세 줄이 주석 처리되어 있어. **모터를 사용하려면 주석을 해제해야 해.**

```cpp
void setup() {
  initDebug();

  // ⬇️ 이 세 줄의 주석을 해제해야 모터 동작!
  // initMotorCommunication();
  // while(!initManipulator()) {}
  // initRGBLED();

  printSelectCommandGuide();
  delay(3000);
}
```

주석 해제 후:

```cpp
void setup() {
  initDebug();
  initMotorCommunication();   // ✅ 주석 해제
  while(!initManipulator()) {}  // ✅ 주석 해제
  initRGBLED();               // ✅ 주석 해제
  printSelectCommandGuide();
  delay(3000);
}
```

### 2단계 — 업로드
Arduino IDE에서 `manageManipulatorPose.ino` 열고 업로드.

### 3단계 — 시리얼 모니터 설정

| 항목 | 값 |
|------|-----|
| 통신 속도 | **115200 baud** (`Debug.h`의 `SERIAL_BAUD_RATE` 기준) |
| 줄 끝 설정 | **Newline (n)** ← 반드시 확인! |

업로드 3초 후 시리얼 모니터에 명령어 가이드가 자동으로 출력돼.

---

## 📟 전체 명령어 체계

```
[COMMAND 모드] ← 기본 상태
  ├── on      : 토크 ON
  ├── off     : 토크 OFF (손으로 팔 이동 가능)
  ├── list    : 저장된 자세 목록 전체 출력
  ├── save    : SAVE 모드 진입
  ├── read    : READ 모드 진입
  ├── remove  : REMOVE 모드 진입
  ├── run     : RUN 모드 진입
  └── help    : 명령어 가이드 출력

각 모드에서:
  ├── help  : 해당 모드 가이드 출력
  └── quit  : COMMAND 모드로 복귀
```

---

## 💾 자세 저장 절차 (실습 핵심)

### Step 1 — 토크 OFF
```
off
```
→ 시리얼 출력: `torque off` → 이제 손으로 팔을 자유롭게 움직일 수 있어.

> 💡 토크가 꺼지면 관절이 무거울 수 있어. 한 손으로 팔을 받치면서 위치를 잡아.

### Step 2 — 원하는 위치로 팔 이동
손으로 팔을 목표 위치(예: INITIAL 대기 자세)로 천천히 이동.

### Step 3 — SAVE 모드 진입
```
save
```
→ 시리얼 출력:
```
save pose
type pose id(1~100), description and enter
   id(uint8_t),description(char[30])
```

### Step 4 — 번호와 이름 입력
```
형식: 번호,이름   ← 쉼표 앞뒤 공백 없이!
```

저장할 자세 목록:

| 입력 | 자세 |
|------|------|
| `1,INITIAL` | 시작/대기 자세 |
| `2,MISSION_1` | 미션지시존 바라보기 |
| `4,SEE_STORAGE` | 적재함 바라보기 |
| `5,GRIP_UPPER` | 위층 집기 준비 |
| `6,GRIP_UPPER_PICK` | 위층 실제 집는 자세 |
| `7,GRIP_LOWER` | 아래층 집기 준비 |
| `8,GRIP_LOWER_PICK` | 아래층 실제 집는 자세 |
| `10,PUT_GREEN` | 초록 블록 배치 위치 |
| `11,PUT_PURPLE` | 보라 블록 배치 위치 |
| `13,PUT_RED` | 빨강 블록 배치 위치 |
| `15,PUT_BLUE` | 파랑 블록 배치 위치 |

→ 저장 성공: **🟢 초록 LED 0.5초 점등** + `save command : pose is saved`
→ 저장 실패: **🔴 빨간 LED 0.5초 점등** + 오류 메시지

### Step 5 — 다음 자세 반복
`quit` 입력 없이 바로 다음 번호를 입력해도 돼. SAVE 모드는 `quit` 전까지 유지돼.

### Step 6 — SAVE 모드 종료
```
quit
```

---

## 📋 저장 목록 확인

```
list
```

출력 예시:
```
1.INITIAL
m1 : 2048, m2 : 1500, m3 : 1800, m4 : 900

2.MISSION_1
m1 : 2048, m2 : 2100, m3 : 1200, m4 : 1600
...
fin
```

---

## 🔍 특정 자세 조회

```
read
↓
15
```

출력 예시:
```
read command : success
   id : 15
   manipulatorMotor1Value : 1900
   manipulatorMotor2Value : 2500
   manipulatorMotor3Value : 950
   manipulatorMotor4Value : 2400
   description : PUT_BLUE
```

---

## ▶️ 저장된 자세 실행 (동작 확인)

### RUN 모드 사용법

```
run
↓
형식 A: 번호,시간           ← 기본
형식 B: 번호,시간,모터1각도  ← 1번 모터 각도 오버라이드
```

**형식 A 예시 — INITIAL 자세로 1000ms 동안 이동:**
```
1,1000
```

**형식 B 예시 — 1번 모터를 -90도로 오버라이드:**
```
1,1000,-90
```

> ⚠️ motor1Angle 범위: **-100 ~ 100도** (이 범위를 넘으면 자동으로 클램핑됨)

저장된 자세 없을 때:
```
run command : there is no data
```
→ **🔴 빨간 LED 0.5초 점등**

---

## 🗑️ 자세 삭제

```
remove
↓
15
```

→ `remove command : pose is removed` + **🟢 초록 LED**

> 삭제는 `isTherePoseData`를 false로만 바꿔. 실제 데이터가 완전히 지워지지는 않아서 같은 번호로 다시 save하면 덮어쓰기 가능.

---

## ✅ 전체 테스트 권장 순서

```
on              ← 토크 ON
run             ← 실행 모드
1,1000          ← INITIAL 확인
2,1000          ← MISSION_1 확인
4,1000          ← SEE_STORAGE 확인
5,1500          ← GRIP_UPPER 확인
6,1500          ← GRIP_UPPER_PICK 확인
5,1000          ← GRIP_UPPER로 복귀
7,1500          ← GRIP_LOWER 확인
8,1500          ← GRIP_LOWER_PICK 확인
7,1000          ← GRIP_LOWER로 복귀
10,1000         ← PUT_GREEN 확인
11,1000         ← PUT_PURPLE 확인
13,1000         ← PUT_RED 확인
15,1000         ← PUT_BLUE 확인
1,1000          ← INITIAL로 최종 복귀
quit            ← RUN 모드 종료
```

---

## ⚠️ 자주 하는 실수

| 실수 | 증상 | 해결 |
|------|------|------|
| 주석 해제 안 함 | 팔이 전혀 안 움직임 | `initMotorCommunication()` 등 3줄 주석 해제 |
| baud rate 틀림 | 시리얼 출력이 깨짐 | 115200으로 변경 |
| 줄끝 설정 틀림 | 명령어 입력해도 반응 없음 | Newline(n)으로 변경 |
| 쉼표 앞뒤 공백 | `save command : wrong parameters count` | `1,INITIAL` (공백 없이) |
| EEPROM 미초기화 | 이상한 자세값 저장됨 | `eeprom_clear` 먼저 실행 |
| 설명 30자 초과 | `description size overflow` | 30자 이내로 줄이기 |
| 저장 안 하고 run | `there is no data` • 🔴 빨간 LED | save 먼저 실행 |

---

## 🔗 EEPROM 주소 계산 방식

```cpp
// Manipulator.h에서
#define MANIPULATOR_POSE_DATA_SIZE  40  // 자세 하나당 40바이트

// 저장 위치 계산
EEPROM.put(40 * (id - 1), manipulatorPose);
// id=1  → 0번지
// id=2  → 40번지
// id=15 → 560번지
```

자세 하나에 저장되는 데이터:

```cpp
typedef struct _ManipulatorPose {
  bool    isTherePoseData;          // 유효 여부 (1바이트)
  uint8_t id;                       // 자세 번호 (1바이트)
  int16_t manipulatorMotor1Value;   // 모터5 위치값 (2바이트)
  int16_t manipulatorMotor2Value;   // 모터6 위치값 (2바이트)
  int16_t manipulatorMotor3Value;   // 모터7 위치값 (2바이트)
  int16_t manipulatorMotor4Value;   // 모터8 위치값 (2바이트)
  char    description[30];          // 설명 텍스트 (30바이트)
} ManipulatorPose;                  // 총 38바이트 (패딩 포함 40바이트)
```

> 💡 **EEPROM은 전원을 꺼도 데이터가 유지돼.** 한 번 저장해두면 대회 코드로 교체해도 그대로 사용 가능.
