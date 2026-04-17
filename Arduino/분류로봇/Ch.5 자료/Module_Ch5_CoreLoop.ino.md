---
title: "🛠️ 모듈 실습 코드 — Module_Ch5_CoreLoop.ino"
parent: "[[Ch.5 — 핵심 변수 연결 구조]]"
source: "Notion"
tags: [arduino, robot, module, ino]
---

# 🛠️ 모듈 실습 코드 — Module_Ch5_CoreLoop.ino

## 📌 이 코드로 할 수 있는 것

- 핵심 루프(슬롯 탐색 → 집기 → 배치)를 **단독으로** 테스트
- `Pixy_List[]`를 직접 수정해서 다양한 시나리오 테스트 가능
- 대회 전체 코드와 동일한 로직 — **모듈화된 검증 코드**

---

## ⚠️ 사전 조건

- Ch.3 EEPROM 자세 저장 **완료** 상태
- Ch.4 SLOT_PSD[] 보정 **완료** 상태
- 로봇이 적재함 앞에 **정렬** 된 상태

---

## 형 확인해야 할 수정 단 위치

```cpp
// ★ 실제 미션존 스캔 결과로 수정
uint8_t Pixy_List[6] = {5, 1, 4, 6, 0, 0};
uint8_t Pixy_B = 4;  // 블록 개수

// ★ 현장 실측값으로 수정
float SLOT_PSD[4] = {17.5, 24.5, 31.0, 35.5};
```

---

## 💻 Arduino 코드 (핵심 로직만 추출)

```cpp
#include "전체 헤더 파일..."

// ★ 실제 미션존 스캔 결과로 수정
uint8_t Pixy_List[6] = {5, 1, 4, 6, 0, 0};
uint8_t Pixy_B = 4;
float SLOT_PSD[4] = {17.5, 24.5, 31.0, 35.5};
#define SP  6
#define UPPER_STANDARD 104
int Put_Position_ByColor[7] = {0, 13, 0, 0, 10, 15, 11};
int pose = 0;

void Grap_blocks(int blockY) {
  bool isUpper = (blockY < UPPER_STANDARD);
  if (isUpper) {
    pose = 5;
    RunManipulatorPoseWithPoseDataInEEPROM(dxl, 5, 1000); // GRIP_UPPER
    RunManipulatorPoseWithPoseDataInEEPROM(dxl, 6, 1000); // GRIP_UPPER_PICK
    CloseGripper(pixy);
    RunManipulatorPoseWithPoseDataInEEPROM(dxl, 5, 1000); // 복귀
  } else {
    pose = 7;
    RunManipulatorPoseWithPoseDataInEEPROM(dxl, 7, 1000); // GRIP_LOWER
    RunManipulatorPoseWithPoseDataInEEPROM(dxl, 8, 1000); // GRIP_LOWER_PICK
    CloseGripper(pixy);
    RunManipulatorPoseWithPoseDataInEEPROM(dxl, 7, 1000);
  }
}

void PUT_ON(int putPos) {
  RunManipulatorPoseWithPoseDataInEEPROM(dxl, putPos, 1000);
  OpenGripper(pixy);
  RunManipulatorPoseWithPoseDataInEEPROM(dxl, pose, 1000);
}

void runCoreLoop() {
  uint8_t count = 0;
  uint8_t findBoxNum = Pixy_B;

  while (findBoxNum > 0) {
    bool foundThisBox = false;
    int retryCount = 0;

    while (!foundThisBox && retryCount < 3) {
      retryCount++;
      for (int slot = 0; slot < 4; slot++) {
        DriveLeftWithSideLeftSensorPID(dxl, SLOT_PSD[slot], 0.2, 1.0, 0.05, 0.2, 50, 100);
        pixy.setLamp(1, 1);
        pixy.ccc.getBlocks();

        int targetColor = Pixy_List[count];
        int targetIndex = -1;
        for (int i = 0; i < pixy.ccc.numBlocks; i++) {
          if (pixy.ccc.blocks[i].m_signature == targetColor) {
            targetIndex = i; break;
          }
        }
        if (targetIndex == -1) continue;

        // 발견!
        foundThisBox = true;
        int y = pixy.ccc.blocks[targetIndex].m_y;
        Grap_blocks(y);
        DriveLeftWithSideLeftSensorPID(dxl, SP, 0.5, 1.0, 0.05, 0.2, 50, 100);
        PUT_ON(Put_Position_ByColor[targetColor]);
        count++; findBoxNum--;
        break;
      }
    }
    if (!foundThisBox) break; // 3회 실패
  }
  pixy.setLamp(0, 0);
}

void setup() {
  // 전체 초기화...
  Serial.println("SW1 버튼으로 핵심 루프 실행");
}

void loop() {
  FL_PSD_Save(); FR_PSD_Save(); SL_PSD_Save(); SL430_PSD_Save();
  if (digitalRead(SW1_PIN) == 0) {
    delay(50);
    runCoreLoop();
    RunManipulatorPoseWithPoseDataInEEPROM(dxl, 1, 1000); // INITIAL 복귀
  }
}
```
