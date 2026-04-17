---
title: "🔄 ADC → cm 방식 전환 가이드"
parent: "[[Ch.4 — PSD 거리값 보정]]"
source: "Notion"
tags: [arduino, robot, psd, adc, conversion]
---

# 🔄 ADC → cm 방식 전환 가이드

ADC 방식 실습(섹션A~E)을 완료한 후, cm 기반 PID 방식으로 전환하는 업그레이드 경로야.

> 💡 **왜 cm 방식이 더 좋은가?**
> ADC 방식은 환경이 바뀌면 목표값을 전부 다시 실측해야 해. cm 방식은 물리적 거리를 직접 입력하므로 직관적이고 환경 변화에 더 강해.

---

## ADC vs cm 방식 비교

| 항목 | ADC 방식 (현재 원본) | cm 방식 (전환 목표) |
|------|----------------------|---------------------|
| 목표값 | `slPSDValue - 518` | `17.5cm` |
| 단위 | 0~1023 (원시값) | cm (물리 단위) |
| 변환 | 없음 | ADC → cm 함수 필요 |
| 제어 | threshold 기반 | PID 기반 |
| 직관성 | 낮음 (숫자 암기) | 높음 (거리 입력) |
| 환경 변화 | 전체 재실측 필요 | cm값만 조정 |
| 코드 예시 | `slValue - 518` | `17.5` |

---

## 전환에 필요한 3가지

### 1️⃣ ADC → cm 변환 함수

PSD 센서는 거리(cm)에 따라 ADC값이 비선형으로 변해. 선형 보간 또는 수식으로 변환해야 해.

```cpp
// Mobilebase.cpp에 추가할 변환 함수 예시
float adcToCm_SL(int16_t adc) {
  // GP2Y0A21YK0F 센서 기준 선형 보간
  // 실측 포인트: (adc, cm) 쌍을 여러 개 측정해서 테이블로 만듦
  if (adc >= 580) return 10.0;
  if (adc >= 420) return 15.0;  // (adc=518) → ~13cm
  if (adc >= 360) return 20.0;
  if (adc >= 310) return 25.0;
  if (adc >= 270) return 30.0;
  if (adc >= 180) return 40.0;
  return 80.0; // 범위 초과
}
```

> ⚠️ 이 값은 예시야. 섹션A에서 각 거리별 ADC값을 직접 측정해서 테이블을 만들어야 해.

**실측 방법:**

| 거리(cm) | SL ADC값 | FL ADC값 | FR ADC값 |
|----------|----------|----------|----------|
| 10cm | ? | ? | ? |
| 15cm | ? | ? | ? |
| 20cm | ? | ? | ? |
| 25cm | ? | ? | ? |
| 30cm | ? | ? | ? |
| 40cm | ? | ? | ? |

### 2️⃣ cm 기반 이동 함수 (Mobilebase.cpp에 추가)

```cpp
// 전방 cm 기반 접근 (GoForwardWithTwoSensors의 cm 버전)
bool ApproachWithFrontSensorsCm(Dynamixel2Arduino dxl,
                                float targetCm,
                                int16_t tolerance) {
  int16_t fl, fr;
  GetValueFromFrontPSDSensors(&fl, &fr);
  float flCm = adcToCm_FL(fl);
  float frCm = adcToCm_FR(fr);

  float error = ((flCm + frCm) / 2.0) - targetCm;
  // PID 계산 후 모터 속도 적용...
}

// 측면 cm 기반 이동 (DriveWithOneSensor의 cm 버전)
bool DriveToSideCm(Dynamixel2Arduino dxl,
                   float targetCm,
                   uint8_t direction) {
  int16_t sl;
  GetValueFromSideLeftPSDSensor(&sl);
  float slCm = adcToCm_SL(sl);
  float error = slCm - targetCm;
  // PID 계산 후 모터 속도 적용...
}
```

### 3️⃣ PID 게인 튜닝

cm 단위로 오차를 계산하면 ADC 단위보다 오차값 자체가 작아져서 게인을 새로 조정해야 해.

```cpp
// ADC 방식: 오차 범위 0~600
// cm 방식:  오차 범위 0~70
// → Kp를 약 10배 크게 설정하는 게 시작점

#define KP_SIDE  3.0   // ADC 방식 대비 큰 값
#define KI_SIDE  0.01
#define KD_SIDE  0.1
```

---

## 단계별 전환 경로

```
[1단계] 현재 (ADC 방식)
  섹션A로 ADC값 실측
  → 섹션B~E로 동작 확인
  → 경진대회 코드 동작 검증

[2단계] 변환 함수 추가
  각 거리(10~40cm)에서 ADC값 실측
  → adcToCm_FL/FR/SL 함수 작성
  → 섹션A에서 cm 출력 확인

[3단계] cm 기반 함수 추가
  ApproachWithFrontSensorsCm() 작성
  DriveToSideCm() 작성
  → 섹션B/C 동작과 결과 비교

[4단계] PID 튜닝
  Kp/Ki/Kd 조정
  → 목표 cm에 정확히 서는지 확인

[5단계] 경진대회 코드 교체
  원본의 ADC 오차 계산 부분을 cm 함수로 교체
  → 목표값을 숫자 대신 cm로 입력
```

---

## 실제 코드 교체 예시

### 전방 접근

```cpp
// ❌ 기존 ADC 방식
while(1) {
  GetValueFromFrontPSDSensors(&fl, &fr);
  if (!GoForwardWithTwoSensors(dxl, fl - 530, fr - 525, 5)) break;
}

// ✅ cm 방식으로 교체
while(1) {
  if (!ApproachWithFrontSensorsCm(dxl, 13.0, 1)) break;
  // 13.0cm → 실측으로 결정
}
```

### 측면 슬롯 이동

```cpp
// ❌ 기존 ADC 방식
while(1) {
  GetValueFromSideLeftPSDSensor(&sl);
  if (!DriveWithOneSensor(dxl, sl - 518, 5, DRIVE_DIRECTION_LEFT)) break;
}

// ✅ cm 방식으로 교체
while(1) {
  if (!DriveToSideCm(dxl, 14.0, DRIVE_DIRECTION_LEFT)) break;
  // 14.0cm → 미션지시존 앞 실측 거리
}
```

---

## ✅ 이해 체크

- [ ] ADC → cm 변환이 왜 필요한지 설명할 수 있다
- [ ] 섹션A로 거리별 ADC값을 측정해서 변환 테이블을 만들 수 있다
- [ ] PID 게인이 단위(ADC vs cm)에 따라 달라지는 이유를 설명할 수 있다
- [ ] ADC 방식 코드를 cm 방식으로 교체할 수 있다
