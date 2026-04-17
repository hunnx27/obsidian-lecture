---
title: "🦴 Ch.3 — EEPROM 자세 저장"
parent: "[[00 - 색깔 블록 분류 로봇 — 경연 핵심 강의]]"
source: "Notion"
tags: [arduino, robot, lecture, eeprom]
---

# 🦴 Ch.3 — EEPROM 자세 저장

> 로봇팔을 원하는 위치로 직접 이동하고, 그 각도를 **EEPROM에 저장**할 수 있다.

---

## EEPROM 자세 저장

```
하나의 자세 = 모터 4개의 각도값 세트

EEPROM 1번 자세 (INITIAL):
  모터5(ID5): 2048
  모터6(ID6): 1500
  모터7(ID7): 1800
  모터8(ID8): 900
↑ 이 4개가 한 세트로 EEPROM에 저장됨
```

주소 계산 방식:

```cpp
EEPROM.put(40 * (id - 1), manipulatorPose);
// 1번 → 0번지, 2번 → 40번지, 15번 → 560번지
```

---

## 저장해야 하는 자세 목록

| ID | 이름 | 언제 쓰이나 |
|----|------|-------------|
| 1 | INITIAL | 시작/끝 대기 자세 |
| 2 | MISSION_1 | 미션지시존 바라보기 |
| 4 | SEE_STORAGE | 적재함 바라보기 |
| 5 | GRIP_UPPER | 위층 집기 준비 |
| 6 | GRIP_UPPER_PICK | 위층 실제 집는 순간 |
| 7 | GRIP_LOWER | 아래층 집기 준비 |
| 8 | GRIP_LOWER_PICK | 아래층 실제 집는 순간 |
| 10 | PUT_2 | 초록 융지 |
| 11 | PUT_3 | 보라 융지 |
| 13 | PUT_5 | 빨강 융지 |
| 15 | PUT_7 | 파랑 융지 |

---

## 저장 절차 (실습)

### 1단계 — 저장용 코드 업로드
제공된 `SavePose.ino` 파일을 Arduino IDE에서 열고 업로드

### 2단계 — 시리얼 모니터 열기

```
속도: 9600
줄끝 설정: Newline (← 이거 꼬듭 확인!)
```

### 3단계 — 토크 OFF

```
off     ← 입력 후 엔터
```

이제 손으로 팔을 자유롭게 움직일 수 있음

### 4단계 — 저장 모드 진입

```
save    ← 엔터
```

### 5단계 — 팔을 원하는 위치로 직접 이동 후 입력

```
1,INITIAL           ← 초록불 = 저장 성공
2,MISSION_1
4,SEE_STORAGE
5,GRIP_UPPER
6,GRIP_UPPER_PICK
7,GRIP_LOWER
8,GRIP_LOWER_PICK
10,PUT_GREEN
11,PUT_PURPLE
13,PUT_RED
15,PUT_BLUE
```

> ⚠️ 빨간불 = 오류 (형식 확인)

### 6단계 — 동작 확인

```
quit    ← save 모드 나가기
on      ← 토크 켜기
run     ← 실행 모드
1,1000  ← 1번 자세를 1000ms 동안 실행
```

할 자세마다 실제로 로봇팔이 움직이는지 눈으로 확인!

### 7단계 — 대회 코드로 교체 업로드

```
SavePose.ino → 대회코드.ino 교체
EEPROM은 전원 꺼도 유지됨 → 다시 저장 불필요
```

---

## 핵심 코드 이해

```cpp
// 저장: 지금 팔 위치를 읽어서 EEPROM에 넣음
void WriteManipulatorPresentPoseToEEPROM(dxl, id, description) {
  pose.motor1 = dxl.readControlTableItem(PRESENT_POSITION, ID5);
  pose.motor2 = dxl.readControlTableItem(PRESENT_POSITION, ID6);
  pose.motor3 = dxl.readControlTableItem(PRESENT_POSITION, ID7);
  pose.motor4 = dxl.readControlTableItem(PRESENT_POSITION, ID8);
  EEPROM.put(40 * (id-1), pose);  // 저장!
}

// 실행: EEPROM에서 꺼내서 모터에 전송
void RunManipulatorPoseWithPoseDataInEEPROM(dxl, id, time) {
  EEPROM.get(40 * (id-1), pose);  // 긺아오기
  syncWrite(pose.motor1, pose.motor2, pose.motor3, pose.motor4, time);
}
```

---

## 🖥️ JSX 시뮬레이션

**파일명:** `robot_flowchart2.jsx`

- `⚙️ 초기화` 단계 클릭 → `pose = INITIAL(1)` 변수 확인
- `✋ 블록 집기` 단계 클릭 → `pose = GRIP_UPPER(5)`, `gripperState = CLOSED` 확인

---

## ✅ 이해 체크

- [ ] 토크 OFF 후 손으로 팔을 움직일 수 있다
- [ ] save 명령어로 자세 저장하고 초록불을 확인할 수 있다
- [ ] run 명령어로 저장된 자세를 실제로 실행하고 동작을 확인할 수 있다
- [ ] EEPROM은 전원을 꺼도 유지된다는 것을 안다

---

### 관련 자료
- Ch3 시뮬레이션 코드 — `Ch3_EEPROMSave.jsx`
- 🛠️ 모듈 실습 코드 — `Module_Ch3_ArmPose.ino`
- Ch.3 — EEPROM 자세 저장 (`manageManipulatorPose.ino` 기반)
- 🎬 시나리오별 실전 명령 예시 — `manageManipulatorPose.ino`
- 💾 EEPROM 백업 & 복원 가이드 — `BackupRestore_EEPROM.ino`
- 💡 팁 모음
- 🦷 `SimplePickPlace.ino` — 집기 & 놓기 단순 테스트
