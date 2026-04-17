---
title: "🔧 실제 에러 해석 가이드 — Ch6b_ErrorGuide.jsx"
parent: "[[Ch.6 — 당일 체크리스트]]"
source: "Notion"
tags: [arduino, robot, simulation, jsx, troubleshooting]
---

# 🔧 실제 에러 해석 가이드 — Ch6b_ErrorGuide.jsx

## 📌 실행 방법

1. `Ch6b_ErrorGuide.jsx` 파일로 저장
2. VSCode Live Server로 실행
3. **증상 클릭** → 원인 + 해결 + 관련 코드 확인
4. 아래 카테고리 필터로 원하는 항목만 보기

---

## 🔧 단골 에러 8개

| 카테고리 | 증상 |
|----------|------|
| 모터/통신 | 시리얼 아무것도 안 뜸 |
| 모터/통신 | Motor count does not match |
| EEPROM | 팔이 전혀 안 움직임 |
| 카메라 | Pixy_B가 0으로 출력됨 |
| 카메라 | 블록 색깔이 섞임 |
| 이동/PSD | 슬롯 엉뚱한 곳 정지 |
| 집기/배치 | 집게가 빗나감 |
| 집기/배치 | 팔이 떨림 |

---

## 📄 핵심 에러 코드 예시

```cpp
// EEPROM 저장 안 된 경우 팔이 안 움직임
ManipulatorPose pose = ReadManipulatorPresentPoseToEEPROM(id);
if (pose.isTherePoseData) {  // ← false이면 실행 안 됨!
  setManipulatorForwardMove(...);
}

// PSD 실측 디버깅
void loop() {
  float dist = SL_PSD_Save();
  Serial.println(dist); // 이 값으로 SLOT_PSD 수정!
}

// Pixy2 감지 디버깅
pixy.ccc.getBlocks();
Serial.print("감지 수: ");
Serial.println(pixy.ccc.numBlocks); // 0이면 문제
```
