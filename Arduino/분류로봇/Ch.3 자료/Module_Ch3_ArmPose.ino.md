---
title: "🛠️ 모듈 실습 코드 — Module_Ch3_ArmPose.ino"
parent: "[[Ch.3 — EEPROM 자세 저장]]"
source: "Notion"
tags: [arduino, robot, module, ino, eeprom]
---

# 🛠️ 모듈 실습 코드 — Module_Ch3_ArmPose.ino

## 📌 이 코드로 할 수 있는 것

- 로봇팔 자세 저장/실행을 **단독으로** 테스트
- `off/on/save/run/list/read/del` 명령어로 작동
- SavePose.ino보다 **간소화된 버전** — 개념 이해에 집중

---

## 🚀 사용 순서

1. `Module_Ch3_ArmPose.ino` 업로드
2. 시리얼 모니터 열기 (9600bps, **줄끝: Newline**)
3. `off` 입력 → 손으로 팔 이동
4. `save` 입력 → `1,INITIAL` 입력
5. `run` 입력 → `1,1000` 입력으로 동작 확인

---

## 📋 명령어 정리

| 명령어 | 동작 |
|--------|------|
| `off` | 토크 OFF (손으로 팔 이동 가능) |
| `on` | 토크 ON |
| `save` | 저장 모드 진입 → `번호,이름` 입력 |
| `run` | 실행 모드 진입 → `번호,시간` 입력 |
| `list` | 저장된 모든 자세 목록 |
| `read,15` | 15번 자세 정보 확인 |
| `del,15` | 15번 자세 삭제 |

---

## 💻 Arduino 코드

```cpp
#include "Debug.h"
#include "Manipulator.h"
#include "Motor.h"
#include "Pins.h"

Dynamixel2Arduino dxl(DXL_SERIAL, DXL_DIR_PIN);

void printPose(ManipulatorPose p) {
  Serial.print("  ID: ");         Serial.println(p.id);
  Serial.print("  설명: ");       Serial.println(p.description);
  Serial.print("  모터1(ID5): ");  Serial.println(p.manipulatorMotor1Value);
  Serial.print("  모터2(ID6): ");  Serial.println(p.manipulatorMotor2Value);
  Serial.print("  모터3(ID7): ");  Serial.println(p.manipulatorMotor3Value);
  Serial.print("  모터 4(ID8): "); Serial.println(p.manipulatorMotor4Value);
}

void setup() {
  Serial.begin(9600);
  InitDebug();
  InitMotorCommunication(dxl);
  while (!InitManipulator(dxl)) { delay(1000); }
  Serial.println("로봇팔 자세 저장 실습 코드");
  PrintManipulatorPoseListFromEEPROM();
}

void loop() {
  if (!Serial.available()) return;
  String input = Serial.readStringUntil('\n');
  input.trim();

  if (input.equalsIgnoreCase("off")) {
    turnOffManipulatorTorque(dxl);
    Serial.println("[완료] 토크 OFF");
  } else if (input.equalsIgnoreCase("on")) {
    turnOnManipulatorTorque(dxl);
    Serial.println("[완료] 토크 ON");
  } else if (input.equalsIgnoreCase("list")) {
    PrintManipulatorPoseListFromEEPROM();
  } else if (input.equalsIgnoreCase("save")) {
    Serial.println("[저장 모드] 번호,이름 입력:");
    while (!Serial.available()) {}
    String param = Serial.readStringUntil('\n'); param.trim();
    int comma = param.indexOf(',');
    if (comma == -1) { Serial.println("[오류] 콤마로 구분해주세요"); return; }
    uint8_t id = param.substring(0, comma).toInt();
    String desc = param.substring(comma + 1);
    WriteManipulatorPresentPoseToEEPROM(dxl, id, desc);
    Serial.println("\n[저장 완료]");
    printPose(ReadManipulatorPresentPoseToEEPROM(id));
  } else if (input.equalsIgnoreCase("run")) {
    Serial.println("[실행 모드] 번호,시간(ms) 입력:");
    while (!Serial.available()) {}
    String param = Serial.readStringUntil('\n'); param.trim();
    int comma = param.indexOf(',');
    uint8_t id = param.substring(0, comma).toInt();
    int32_t t  = param.substring(comma + 1).toInt();
    ManipulatorPose p = RunManipulatorPoseWithPoseDataInEEPROM(dxl, id, t);
    if (p.isTherePoseData) { Serial.println("\n[실행 완료]"); printPose(p); }
    else { Serial.println("[오류] 저장된 자세 없음"); }
  } else if (input.startsWith("read,")) {
    uint8_t id = input.substring(5).toInt();
    ManipulatorPose p = ReadManipulatorPresentPoseToEEPROM(id);
    if (p.isTherePoseData) printPose(p);
    else Serial.println("[오류] 저장 없음");
  } else if (input.startsWith("del,")) {
    uint8_t id = input.substring(4).toInt();
    RemoveManipulatorPresentPoseFromEEPROM(id);
    Serial.print("[삭제 완료] "); Serial.print(id); Serial.println("번");
  }
}
```
