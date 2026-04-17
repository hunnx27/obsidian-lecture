---
title: "📊 PSD 노이즈 필터 시뮬레이터 — Ch4b_PSDNoiseFilter.jsx"
parent: "[[Ch.4 — PSD 거리값 보정]]"
source: "Notion"
tags: [arduino, robot, simulation, jsx, psd, filter]
---

# 📊 PSD 노이즈 필터 시뮬레이터 — Ch4b_PSDNoiseFilter.jsx

## 📌 실행 방법

1. `Ch4b_PSDNoiseFilter.jsx` 파일로 저장
2. VSCode Live Server로 실행
3. 3개 탭 활용:
	- **📊 실시간 시뮬** — 기준거리/노이즈 조절 → RAW vs 평균 라인 비교
	- **🧮 원리 설명** — 이동평균 필터가 뭔지
	- **💻 실제 코드** — 코드에서 `readings[]`, `total[]`, `average[]` 동작 방식

---

## 🤔 왜 PSD 값이 흔들리나?

- ⚠️ 형광등 깜박임 → 적외선 간섭
- ⚠️ 바닥 반사율 차이 → 지점마다 오차
- ⚠️ 로봇 진동 → 센서 각도 미세 변화
- ⚠️ ADC 변환 오차 → 디지털 노이즈

---

## 💡 이동평균 필터란?

연속으로 N개의 값을 저장해두고 그 평균을 실제 거리로 사용하는 방법이야.

```
측정값: [24.1, 27.3, 24.8, 25.2, 23.9]
평균: (24.1+27.3+24.8+25.2+23.9) / 5
= 25.06cm ← 안정적!
```

---

## 💻 실제 Arduino 코드 (이미 대회 코드에 구현됨)

```cpp
// 전역 변수
float readings[PSD_CNT][NUM_READINGS]; // 최근 10개 저장
int16_t readIndex = 0;                 // 저장 위치
float total[PSD_CNT];                  // 합계
float average[PSD_CNT];                // 평균 (실제 거리값)

// SL_PSD_Save() 내부 동작
float SL_PSD_Save() {
  float newVal = analogRead(PIN_SIDE_LEFT_PSD);
  total[SL] -= readings[SL][readIndex]; // 오래된 값 빼기
  readings[SL][readIndex] = newVal;     // 새 값 저장
  total[SL] += newVal;                  // 새 값 더하기
  readIndex = (readIndex+1) % NUM_READINGS; // 순환
  average[SL] = total[SL] / NUM_READINGS;  // 평균
  return average[SL]; // 학 값 반환
}
```

---

## 📄 JSX 코드 (시뮬레이터 핵심)

```jsx
import { useState, useEffect, useRef } from "react";

const NUM_READINGS = 10;

function genRaw(base, noiseLevel) {
  return base + (Math.random() - 0.5) * noiseLevel * 2;
}

export default function App() {
  const [tab, setTab] = useState(0);
  const [running, setRunning] = useState(false);
  const [base, setBase] = useState(25);
  const [noise, setNoise] = useState(3);
  const [rawHistory, setRawHistory] = useState([]);
  const [avgHistory, setAvgHistory] = useState([]);
  const [window, setWindow] = useState([]);
  const intervalRef = useRef(null);

  useEffect(() => {
    if (running) {
      intervalRef.current = setInterval(() => {
        const raw = parseFloat(genRaw(base, noise).toFixed(2));
        setRawHistory(h => [...h.slice(-39), raw]);
        setWindow(w => {
          const next = [...w.slice(-(NUM_READINGS - 1)), raw];
          const avg = parseFloat((next.reduce((a, b) => a + b, 0) / next.length).toFixed(2));
          setAvgHistory(h => [...h.slice(-39), avg]);
          return next;
        });
      }, 200);
    } else { clearInterval(intervalRef.current); }
    return () => clearInterval(intervalRef.current);
  }, [running, base, noise]);

  // ... 3탭 렌더링 (실시간 시뮬 / 원리 설명 / 실제 코드)
  // 본문 렌더링은 원본 페이지 참고
}
```
