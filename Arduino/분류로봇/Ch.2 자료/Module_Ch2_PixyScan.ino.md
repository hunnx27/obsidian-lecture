---
title: "🛠️ 모듈 실습 코드 — Module_Ch2_PixyScan.ino"
parent: "[[Ch.2 — Pixy2 카메라 세팅]]"
source: "Notion"
tags: [arduino, robot, module, ino, pixy2]
---

# 🛠️ 모듈 실습 코드 — Module_Ch2_PixyScan.ino

## 📌 이 코드로 할 수 있는 것

- Pixy2 블록 스캔 + 버블정렬을 **단독으로** 테스트
- SW1 버튼 누를 때마다 스캔 결과를 시리얼 모니터에 출력
- 색깔별 sig 번호 + X좌표 + 정렬 순서 확인

---

## 🚀 사용 순서

1. `Module_Ch2_PixyScan.ino` 업로드
2. 시리얼 모니터 열기 (9600bps)
3. 블록 4개를 카메라 앞에 배치
4. **SW1 버튼** 누르면 스캔 결과 출력
5. 출력된 정렬 순서가 왼→오른 맞는지 확인

---

## 📋 시리얼 출력 예시

```
[버튼 입력] 스캔 시작...
====== 스캔 결과 (왼→오른) ======
  [0] sig=5  x=55   (파랑)
  [1] sig=4  x=85   (초록)
  [2] sig=6  x=142  (보라)
  [3] sig=1  x=210  (빨강)
총 블록 수: 4
================================
```

---

## 💻 Arduino 코드

```cpp
#include "Camera.h"
#include "Debug.h"
#include "Pins.h"

Pixy2SPI_SS pixy;
#define MAX_BLOCKS 6
uint8_t Pixy_List[MAX_BLOCKS] = {0,};
int     num_x[MAX_BLOCKS]     = {0,};
uint8_t Pixy_B = 0;

void Pixy_xNUM_OneShot() {
  pixy.ccc.getBlocks();
  Pixy_B = pixy.ccc.numBlocks;
  if (Pixy_B == 0) { Serial.println("[결과] 감지된 블록 없음"); return; }
  if (Pixy_B > MAX_BLOCKS) Pixy_B = MAX_BLOCKS;

  for (int i = 0; i < Pixy_B; i++) {
    num_x[i]     = pixy.ccc.blocks[i].m_x;
    Pixy_List[i] = pixy.ccc.blocks[i].m_signature;
  }

  // 버블정렬 (왼→오른)
  for (int i = 0; i < Pixy_B - 1; i++) {
    for (int j = 0; j < Pixy_B - 1 - i; j++) {
      if (num_x[j] > num_x[j + 1]) {
        int tx = num_x[j];     num_x[j] = num_x[j+1];     num_x[j+1] = tx;
        int ts = Pixy_List[j]; Pixy_List[j] = Pixy_List[j+1]; Pixy_List[j+1] = ts;
      }
    }
  }

  Serial.println("====== 스캔 결과 (왼→오른) ======");
  for (int i = 0; i < Pixy_B; i++) {
    Serial.print("  ["); Serial.print(i); Serial.print("] sig=");
    Serial.print(Pixy_List[i]); Serial.print("  x=");
    Serial.print(num_x[i]); Serial.print("  (");
    switch(Pixy_List[i]) {
      case 1: Serial.print("빨강"); break;
      case 4: Serial.print("초록"); break;
      case 5: Serial.print("파랑"); break;
      case 6: Serial.print("보라"); break;
      default: Serial.print("미지정"); break;
    }
    Serial.println(")");
  }
  Serial.print("총 블록 수: "); Serial.println(Pixy_B);
}

void setup() {
  Serial.begin(9600);
  InitDebug(); InitPin();
  pixy.init(); InitPixy(pixy);
  pixy.setLamp(1, 1);
  Serial.println("SW1 버튼을 누를 때마다 스캔합니다.");
}

void loop() {
  if (digitalRead(SW1_PIN) == 0) {
    delay(50);
    Serial.println("\n[버튼 입력] 스캔 시작...");
    Pixy_xNUM_OneShot();
    delay(500);
  }
  static unsigned long lastPrint = 0;
  if (millis() - lastPrint > 1000) {
    pixy.ccc.getBlocks();
    Serial.print("실시간 감지 수: "); Serial.println(pixy.ccc.numBlocks);
    lastPrint = millis();
  }
}
```
