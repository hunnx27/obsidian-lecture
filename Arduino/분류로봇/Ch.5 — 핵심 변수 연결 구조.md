---
title: "🔗 Ch.5 — 핵심 변수 연결 구조"
parent: "[[00 - 색깔 블록 분류 로봇 — 경연 핵심 강의]]"
source: "Notion"
tags: [arduino, robot, lecture, code-structure]
---

# 🔗 Ch.5 — 핵심 변수 연결 구조

> 코드에서 **변수 4개**가 어떻게 연결되는지 설명할 수 있다.

---

## 4개 핵심 변수

```
Pixy_List[]            → 블록 처리 순서 결정
SLOT_PSD[]             → 슬롯 이동 거리 결정
Put_Position_ByColor[] → 색깔별 배치 위치 결정
EEPROM 자세번호        → 실제 팔 동작 결정
```

> ⚠️ **이 네 가지 중 하나라도 잘못되면 전체 미션 실패!**

---

## 1. Pixy_List[] — 처리 순서

```cpp
// Pixy_xNUM_OneShot() 호출 후 결과
uint8_t Pixy_List[6] = {5, 4, 6, 1}; // 파랑, 초록, 보라, 빨강 순서

// 핵심 루프에서
int targetColor = Pixy_List[count]; // count=0 → sig5(파랑)
```

비유: "쾔상에서 왼쪽부터 차레대로 놓인 순서를 엄격히 지킴다"

---

## 2. SLOT_PSD[] — 슬롯 위치

```cpp
float SLOT_PSD[4] = {17.5, 24.5, 31.0, 35.5};

// 루프에서
for (int slot = 0; slot <= NUM_SLOTS; slot++) {
    DriveLeftWithSideLeftSensorPID(dxl, SLOT_PSD[slot], ...);
    // 이 거리에 도달하면 정지 → 카메라로 스캔
}
```

비유: "에스컈레이터가 1층, 2층, 3층, 4층 앞에 서는 것"

---

## 3. Put_Position_ByColor[] — 색깔별 배치

```cpp
int Put_Position_ByColor[7] = {
  0,   // [0] 미사용
  13,  // [1] 빨강  → EEPROM 13번 자세
  0,   // [2] 미사용
  0,   // [3] 미사용
  10,  // [4] 초록  → EEPROM 10번 자세
  15,  // [5] 파랑  → EEPROM 15번 자세
  11   // [6] 보라  → EEPROM 11번 자세
};

// 사용할 때
int putPos = Put_Position_ByColor[targetColor]; // 색깔번호가 바로 인덱스!
PUT_ON(putPos);
```

비유: "색깔 번호를 열쉬로 주소록 열어보는 것"

---

## 4. EEPROM 자세번호 — 실제 팔 동작

```cpp
void PUT_ON(int putPos) {         // putPos = 15
    // EEPROM 15번 긺아서 모터 4개 이동
    RunManipulatorPoseWithPoseDataInEEPROM(dxl, putPos, 1000);
    OpenGripper(pixy);            // 보타리 내려놓기
    RunManipulatorPoseWithPoseDataInEEPROM(dxl, pose, 1000); // 복귀
}
```

---

## 전체 연결도

```
카메라가 파랑 블록을 발견
    ↓
targetColor = 5
    ↓
Put_Position_ByColor[5] = 15
    ↓
RunManipulatorPoseWithPoseDataInEEPROM(dxl, 15, 1000)
    ↓
EEPROM 560번지에서 모터값 4개 긺아서
모터에 전송 → 파랑 배치위치로 이동 → 블록 내려놓기
```

---

## 🖥️ JSX 시뮬레이션

**파일명:** `robot_deeptab.jsx` → 탭2 전체 진행

- `다음 ▶` 버튼으로 단계별 진행
- 각 단계에서 변수가 어떻게 바뀌는지 오른쪽 메모리 패널에서 확인

---

## ✅ 이해 체크

- [ ] 4개 변수에 어떤 것이 있는지 말할 수 있다
- [ ] 하나가 장못되면 전체가 실패하는 이유를 설명할 수 있다
- [ ] `Put_Position_ByColor[5] = 15` 가 의미하는 것을 설명할 수 있다

---

### 관련 자료
- Ch5 시뮬레이션 코드 — `Ch5_VariableConnection.jsx`
- 🛠️ 모듈 실습 코드 — `Module_Ch5_CoreLoop.ino`
