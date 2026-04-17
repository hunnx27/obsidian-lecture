---
title: "💾 EEPROM 백업 & 복원 가이드 — BackupRestore_EEPROM.ino"
parent: "[[Ch.3 — EEPROM 자세 저장]]"
source: "Notion"
tags: [arduino, robot, eeprom, backup, restore]
---

# 💾 EEPROM 백업 & 복원 가이드 — BackupRestore_EEPROM.ino

## 💡 이게 필요한 상황

- 자세 저장 완료 후 연습을 위해 EEPROM을 초기화하고 싶을 때
- 다른 팀의 로봇에 동일한 자세를 복사할 때
- 실수로 EEPROM을 실행 후 원상복구할 때

---

## 📦 필요한 파일

| 파일 | 역할 |
|------|------|
| `BackupRestore_EEPROM.ino` | 메인 파일 |
| `Debug.h`, `Motor.h`, `Manipulator.h`, `RGBLED.h`, `Pins.h` | 헤더 파일 |

> ⚠️ `manageManipulatorPose.ino`과 달리 이 코드는 **주석 해제 필요 없이** 바로 동작해.

---

## 📝 명령어 정리

| 명령어 | 하는 일 |
|--------|---------|
| `backup` | EEPROM 전체 자세를 복원용 형식으로 출력 |
| `restore` | 모터값 직접 입력해서 EEPROM에 복원 |
| `list` | 현재 저장 목록 출력 |
| `clear` | 전체 삭제 (연습 전 초기화) |
| `help` | 가이드 출력 |

---

## 🎬 시나리오 1 — 연습 전 백업하기

### Step 1 — `BackupRestore_EEPROM.ino` 업로드
### Step 2 — 시리얼 모니터 열기 (115200 / Newline)
### Step 3 — backup 명령 입력

```
[입력] backup

[출력]
====== EEPROM 백업 시작 ======
아래 내용을 복사해서 보관하세요.
복원 시 restore 명령어로 아래 줄을 하나씩 입력하면 됩니다.
------------------------------
1,2048,1500,1800,900,INITIAL
2,2048,2100,1200,1600,MISSION_1
4,2048,1800,1400,2000,SEE_STORAGE
5,2048,2400,900,2200,GRIP_UPPER
6,2048,2600,700,2400,GRIP_UPPER_PICK
7,2048,1900,1600,1400,GRIP_LOWER
8,2048,2100,1400,1600,GRIP_LOWER_PICK
10,1600,2200,1100,2100,PUT_GREEN
11,1700,2300,1050,2200,PUT_PURPLE
13,1800,2400,1000,2300,PUT_RED
15,1900,2500,950,2400,PUT_BLUE
==============================
```

### Step 4 — 이 출력내용을 **복사해서 따로 보관**

```
방법 A: 메모장/노션에 붙여넣기
방법 B: 텍스트 파일로 저장 (backup.txt)
```

---

## 🎬 시나리오 2 — 연습 중 EEPROM 초기화

```
[입력] clear

[출력] [경고] 모든 자세를 삭제합니다. 5초 후 실행...
       취소하려면 리셋 버튼을 누르세요.

(5초 대기)

[출력] [완료] 모든 자세 삭제됨.   🟢

[확인]
list
→ (출력 없음 = 클리어 성공)
```

> ⚠️ **5초 안에 리셋을 누르면 취소** 돼. 실수 입력 시 리셋으로 안전하게 취소 가능.

---

## 🎬 시나리오 3 — 연습 후 원상복구

### Step 1 — restore 모드 진입

```
[입력] restore

[출력] [복원 모드] 아래 형식으로 입력하고 엔터:
  id,m1,m2,m3,m4,description
  예) 1,2048,1500,1800,900,INITIAL
  quit 입력 시 복원 모드 종료
```

### Step 2 — 저장해둔 백업 내용을 하나씩 입력

```
[입력] 1,2048,1500,1800,900,INITIAL          🟢
[입력] 2,2048,2100,1200,1600,MISSION_1       🟢
[입력] 4,2048,1800,1400,2000,SEE_STORAGE     🟢
[입력] 5,2048,2400,900,2200,GRIP_UPPER       🟢
[입력] 6,2048,2600,700,2400,GRIP_UPPER_PICK  🟢
[입력] 7,2048,1900,1600,1400,GRIP_LOWER      🟢
[입력] 8,2048,2100,1400,1600,GRIP_LOWER_PICK 🟢
[입력] 10,1600,2200,1100,2100,PUT_GREEN      🟢
[입력] 11,1700,2300,1050,2200,PUT_PURPLE     🟢
[입력] 13,1800,2400,1000,2300,PUT_RED        🟢
[입력] 15,1900,2500,950,2400,PUT_BLUE        🟢
```

### Step 3 — 복원 모드 종료
```
[입력] quit
```

### Step 4 — 확인
```
[입력] list  → 11개 자세 모두 있으면 복원 성공!
```

### Step 5 — `manageManipulatorPose.ino`로 교체 후 run으로 동작 확인

---

## 🎬 시나리오 4 — 다른 팀 로봇에 자세 복사

```
로봇A에서:
  BackupRestore_EEPROM.ino 업로드
  backup 입력 → 출력된 값들 복사

로봇B에서:
  BackupRestore_EEPROM.ino 업로드
  restore 입력
  복사한 줄들 하나씩 입력
  → 로봇A와 동일한 자세 복원 완료!
```

---

## ⚠️ 주의사항

| 항목 | 내용 |
|------|------|
| 형식 엄수 | 쉼표 앞뒤 공백 없이: `1,2048,1500,1800,900,INITIAL` |
| 범위 이탈 | 모터값은 MIN/MAX 범위 내여야 함 (Manipulator.h 참조) |
| description | 1~30자, 한글 사용 가능하나 영문자 권장 |
| clear 취소 | 5초 안에 리셋 스위치 누르면 취소됨 |

---

## 🔄 전체 흐름 요약

```
[1단계] 연습 전
  BackupRestore_EEPROM.ino 업로드
  backup → 출력 복사 보관

[2단계] 연습 중
  clear → 5초 대기 → EEPROM 비움
  manageManipulatorPose.ino 업로드
  자유롭게 저장/수정/실습

[3단계] 연습 후
  BackupRestore_EEPROM.ino 다시 업로드
  restore → 복사한 값들 하나씩 입력
  list로 확인
  manageManipulatorPose.ino로 교체 업로드
  run으로 실제 동작 확인
```
