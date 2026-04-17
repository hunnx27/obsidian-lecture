---
title: "🎬 시나리오별 실전 명령 예시 — manageManipulatorPose.ino"
parent: "[[Ch.3 — EEPROM 자세 저장]]"
source: "Notion"
tags: [arduino, robot, eeprom, scenarios]
---

# 🎬 시나리오별 실전 명령 예시 — manageManipulatorPose.ino

> 시리얼 모니터 설정: **115200 baud / Newline(n)**
> 코드에서 3줄 주석 해제 후 업로드 필수!

---

## 시나리오 1 — 처음 자세 저장하기

**상황:** 아무것도 저장 안 된 상태에서 INITIAL 자세부터 저장

```
[시리얼 모니터 출력]
type command and enter
   "on" to turn on manipulator motor torque
   "off" to turn off manipulator motor torque
   "list" to show manipulator's pose list
   "save" to save current manipulator's pose
   ...
```

**① 토크 OFF — 손으로 팔 움직이기 위해**
```
[입력] off
[출력] torque off
```

**② 팔을 손으로 INITIAL 위치(안전 대기 자세)로 이동**

**③ SAVE 모드 진입**
```
[입력] save
[출력] save pose
       type pose id(1~100), description and enter
       id(uint8_t),description(char[30])
```

**④ 번호와 이름 입력**
```
[입력] 1,INITIAL
[출력] save command : pose is saved
       🟢 초록 LED 0.5초 점멸
```

**⑤ 같은 방식으로 나머지 자세 연속 저장**
```
[입력] 2,MISSION_1            🟢
[입력] 4,SEE_STORAGE          🟢
[입력] 5,GRIP_UPPER           🟢
[입력] 6,GRIP_UPPER_PICK      🟢
[입력] 7,GRIP_LOWER           🟢
[입력] 8,GRIP_LOWER_PICK      🟢
[입력] 10,PUT_GREEN           🟢
[입력] 11,PUT_PURPLE          🟢
[입력] 13,PUT_RED             🟢
[입력] 15,PUT_BLUE            🟢
```

**⑥ SAVE 모드 종료**
```
[입력] quit
```

---

## 시나리오 2 — 저장 목록 확인하기

```
[입력] list
[출력]
1.INITIAL
m1 : 2048, m2 : 1500, m3 : 1800, m4 : 900

2.MISSION_1
m1 : 2048, m2 : 2100, m3 : 1200, m4 : 1600

4.SEE_STORAGE
m1 : 2048, m2 : 1800, m3 : 1400, m4 : 2000

5.GRIP_UPPER
m1 : 2048, m2 : 2400, m3 : 900, m4 : 2200

6.GRIP_UPPER_PICK
m1 : 2048, m2 : 2600, m3 : 700, m4 : 2400

7.GRIP_LOWER
m1 : 2048, m2 : 1900, m3 : 1600, m4 : 1400

8.GRIP_LOWER_PICK
m1 : 2048, m2 : 2100, m3 : 1400, m4 : 1600

10.PUT_GREEN
m1 : 1600, m2 : 2200, m3 : 1100, m4 : 2100

11.PUT_PURPLE
m1 : 1700, m2 : 2300, m3 : 1050, m4 : 2200

13.PUT_RED
m1 : 1800, m2 : 2400, m3 : 1000, m4 : 2300

15.PUT_BLUE
m1 : 1900, m2 : 2500, m3 : 950, m4 : 2400

fin
```

---

## 시나리오 3 — 저장된 자세 실행해서 동작 확인

```
[입력] on             → 토크 ON
[입력] run            → RUN 모드 진입
[입력] 1,1000         → INITIAL 위치로 이동
[입력] 5,1500         → GRIP_UPPER 위치로 이동
[입력] 6,1500         → GRIP_UPPER_PICK 위치로 이동
[입력] 5,1000         → GRIP_UPPER로 복귀
[입력] 7,1500         → GRIP_LOWER
[입력] 8,1500         → GRIP_LOWER_PICK
[입력] 7,1000         → GRIP_LOWER로 복귀
[입력] 10,1000        → PUT_GREEN
[입력] 13,1000        → PUT_RED
[입력] 15,1000        → PUT_BLUE
[입력] 1,1000         → INITIAL로 최종 복귀
[입력] quit
```

---

## 시나리오 4 — 자세가 잘못 저장됐을 때 덮어쓰기

```
[입력] off
(팔을 정확한 PUT_BLUE 위치로 다시 이동)
[입력] save
[입력] 15,PUT_BLUE        🟢 (덮어쓰기됨)
[입력] quit
[입력] on
[입력] run
[입력] 15,1000            (바뀐 위치로 가는지 재확인)
[입력] quit
```

---

## 시나리오 5 — 특정 자세 정보 조회

```
[입력] read
[입력] 15
[출력] read command : success
       id : 15
       manipulatorMotor1Value : 1900
       manipulatorMotor2Value : 2500
       manipulatorMotor3Value : 950
       manipulatorMotor4Value : 2400
       description : PUT_BLUE
       🟢 초록 LED
[입력] quit
```

---

## 시나리오 6 — 잘못 저장된 자세 삭제

```
[입력] remove
[입력] 3
[출력] remove command : pose is removed  🟢
[입력] quit
[입력] list   (3번이 사라진 것 확인)
```

---

## 시나리오 7 — 오류 상황 & 해결법

**A: 형식 오류 (쉼표 앞뒤 공백)**
```
[입력] 1, INITIAL   ← 쉼표 뒤 공백!
[출력] save command : wrong parameters count  🔴
[해결] 1,INITIAL    ← 공백 없이 입력
```

**B: 존재하지 않는 번호 실행**
```
[입력] 99,1000
[출력] run command : there is no data  🔴
[해결] list로 저장된 번호 확인
```

**C: 설명이 30자 초과**
```
[입력] 1,THIS_IS_TOO_LONG_DESCRIPTION_FOR_EEPROM
[출력] save command : description size overflow  🔴
[해결] 30자 이내로 줄이기
```

**D: 팔이 전혀 안 움직임**
```
[원인] setup()의 3줄이 아직 주석 처리됨
[해결]
  initMotorCommunication();   // 주석 해제
  while(!initManipulator()) {} // 주석 해제
  initRGBLED();               // 주석 해제
  → 재업로드
```

---

## ✅ 완료 기준 체크

- [ ] 11개 자세 모두 `🟢 초록 LED` 확인
- [ ] `list` 명령으로 11개 모두 출력 확인
- [ ] `run`으로 각 자세 실제 동작 눈으로 확인
- [ ] 대회 코드로 교체 업로드
