---
title: "🛠️ 모듈 실습 코드 — Module_Ch4_PSDTest.ino"
parent: "[[Ch.4 — PSD 거리값 보정]]"
source: "Notion"
tags: [arduino, robot, module, ino, psd]
---

# 🛠️ 모듈 실습 코드 — Module_Ch4_PSDTest.ino

## 📌 이 코드로 할 수 있는 것

- PSD 거리값 실시간 출력 (500ms마다)
- 시리얼 명령어(`1~4`)로 슬롯별 이동 테스트
- `s` 명령으로 현재 SLOT_PSD 설정값 확인

---

## 🚀 사용 순서

1. **`Module_Ch4_PSDTest.ino`** 업로드
2. 시리얼 모니터 열기 (9600bps)
3. PSD 값이 실시간으로 출력됨
4. 로봇을 해당 슬롯 앞에 수동 위치
5. `p` 입력 → PSD값 확인 → `SLOT_PSD[]`에 반영
6. `1`~`4` 입력해서 슬롯 이동 테스트

---

## 꼭 확인해야 할 수정 위치

```cpp
// ★ 현장에서 실측한 값으로 수정 ★
float SLOT_PSD[4] = {17.5, 24.5, 31.0, 35.5};
//                   슬롯1  슬롯2  슬롯3  슬롯4
```

---

## 💻 Arduino 코드

```cpp
#include "PSD.h"
#include "MobileBase.h"
#include "Debug.h"
#include "Pins.h"

Dynamixel2Arduino dxl(DXL_SERIAL, DXL_DIR_PIN);

// ★ 현장에서 실측한 값으로 수정
float SLOT_PSD[4] = {17.5, 24.5, 31.0, 35.5};

float readings[PSD_CNT][NUM_READINGS];
int16_t readIndex = 0;
float total[PSD_CNT], average[PSD_CNT];

void printAllPSD() {
  float sl = SL_PSD_Save();
  Serial.print("SL="); Serial.print(sl, 1);
  Serial.print("  FL="); Serial.print(FL_PSD_Save(), 1);
  Serial.print("  FR="); Serial.println(FR_PSD_Save(), 1);
}

void moveToSlot(int idx) {
  Serial.print("\n[이동] 슬롯"); Serial.print(idx+1);
  Serial.print(" 목표="); Serial.print(SLOT_PSD[idx]); Serial.println("cm");
  DriveLeftWithSideLeftSensorPID(dxl, SLOT_PSD[idx], 0.2, 1.0, 0.05, 0.2, 50, 100);
  Serial.print("[완료] 현재 SL="); Serial.print(SL_PSD_Save(), 1); Serial.println("cm");
}

void setup() {
  Serial.begin(9600);
  InitDebug(); InitPSD();
  InitMotorCommunication(dxl);
  while (!InitMobilebase(dxl)) { delay(1000); }
  Serial.println("PSD 실측 코드 | 명령어: 1~4=슬롯이동 p=PSD확인 s=설정확인");
}

void loop() {
  static unsigned long lastPrint = 0;
  if (millis() - lastPrint > 500) { printAllPSD(); lastPrint = millis(); }

  if (Serial.available()) {
    char cmd = Serial.read();
    if      (cmd=='1') moveToSlot(0);
    else if (cmd=='2') moveToSlot(1);
    else if (cmd=='3') moveToSlot(2);
    else if (cmd=='4') moveToSlot(3);
    else if (cmd=='p'||cmd=='P') { Serial.println("\n[현재 PSD]"); printAllPSD(); }
    else if (cmd=='s'||cmd=='S') {
      for (int i=0;i<4;i++) {
        Serial.print("  SLOT_PSD["); Serial.print(i);
        Serial.print("] = "); Serial.println(SLOT_PSD[i]);
      }
    }
  }
}
```
