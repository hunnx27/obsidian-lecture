---
title: "📷 Ch.2 — Pixy2 카메라 세팅"
parent: "[[00 - 색깔 블록 분류 로봇 — 경연 핵심 강의]]"
source: "Notion"
tags: [arduino, robot, lecture, pixy2]
---

# 📷 Ch.2 — Pixy2 카메라 세팅

> Pixy2 카메라가 4개 블록을 **정확히** 구분할 수 있도록 세팅한다.

---

## 핵심 원리

카메라는 블록의 색깔을 **시그니처(Signature) 번호**로 구분한다.

| 시그니처 | 색깔 | 코드에서 |
|----------|------|----------|
| sig 1 | 빨강 | `Pixy_List[i] == 1` |
| sig 4 | 초록 | `Pixy_List[i] == 4` |
| sig 5 | 파랑 | `Pixy_List[i] == 5` |
| sig 6 | 보라 | `Pixy_List[i] == 6` |

---

## PixyMon 세팅 순서

### 1단계 — 조명 조건 설정

- **경연장과 동일한 조명**에서 학습
- 자연광(3습) 또는 현장 세팅에서 실습
- Auto White Balance, Auto Exposure **ON**

### 2단계 — 시그니처 학습

```
PixyMon 실행
  ↓
Action → Set Signature 1 선택
  ↓
블록의 밝은 면만 드래그
(그늘지는 부분 제외! 이게 핵심)
  ↓
4개 색깔 반복
```

> ⚠️ **그늘지는 부분을 포함하면 안 됨!**
> 밝은 면만 선택해야 다른 색과 구분이 잘 됨

### 3단계 — Range 값 조정

```
Range 시작값: 8.0
  ↓
0.5씩 낮춰가면서 확인
  ↓
색깔끌리 겹치지 않는 지점에서 멈춰!
```

예시:

| Range | 상태 |
|-------|------|
| 8.0 | 색깔 감지 잘 됨 |
| 4.0 | 보통 |
| 1.5 | 타이트하지만 반응 덕어짘 |

### 4단계 — Arduino 필터링 (Arduino 코드)

```cpp
// 블록 크기로 노이즈 제거
if (block.m_width > 30 && block.m_height > 30) {
    // 실제 블록만 통과
}
```

---

## 카메라 밝기 기준값

| 환경 | Brightness 값 |
|------|---------------|
| 어두움 | ~140 |
| 보통 | ~120 |
| 밝음 | ~100 |

현장에서 자동으로 조절되지만, **불일치하면 PixyMon에서 수동 조정**

---

## 학습 실습 코드

```cpp
// 다음 코드를 Arduino에 업로드하고
// 시리얼 모니터로 범력 확인
void loop() {
    pixy.ccc.getBlocks();
    Serial.print("감지된 블록 수: ");
    Serial.println(pixy.ccc.numBlocks);

    for (int i = 0; i < pixy.ccc.numBlocks; i++) {
        Serial.print("sig=");
        Serial.print(pixy.ccc.blocks[i].m_signature);
        Serial.print(", x=");
        Serial.print(pixy.ccc.blocks[i].m_x);
        Serial.print(", w=");
        Serial.println(pixy.ccc.blocks[i].m_width);
    }
    delay(500);
}
```

실행 후 시리얼 모니터에서:
- 모든 블록이 sig만다 정확히 나오는지 확인
- 다른 색깔이 섞이는지 확인 (교차 오염 확인)

---

## 🖥️ JSX 시뮬레이션

**파일명:** `robot_deeptab.jsx` → 탭1 (📷 미션존 블록 식별)

- 버블정렬이 한 단계씩 진행되는 것 확인
- `num_x[]`와 `Pixy_List[]`가 동시에 교환되는 것 확인
- 정렬 완료 후 순서가 왜 중요한지 이해

---

## ✅ 이해 체크

- [ ] PixyMon에서 밝은 면만 드래그해서 sig 학습할 수 있다
- [ ] Range값을 대답하는 트레이드오프를 설명할 수 있다
- [ ] Arduino 필터링 코드를 작성하고 실행할 수 있다

---

### 관련 자료
- Ch2 시뮬레이션 코드 — `Ch2_Pixy2Setup.jsx`
- 🛠️ 모듈 실습 코드 — `Module_Ch2_PixyScan.ino`
