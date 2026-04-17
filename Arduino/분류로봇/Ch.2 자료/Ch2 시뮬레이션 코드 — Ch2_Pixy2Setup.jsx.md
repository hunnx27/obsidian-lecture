---
title: "💻 Ch2 시뮬레이션 코드 — Ch2_Pixy2Setup.jsx"
parent: "[[Ch.2 — Pixy2 카메라 세팅]]"
source: "Notion"
tags: [arduino, robot, simulation, jsx, pixy2]
---

# 💻 Ch2 시뮬레이션 코드 — Ch2_Pixy2Setup.jsx

## 📌 실행 방법

1. 아래 코드를 `Ch2_Pixy2Setup.jsx` 파일로 저장
2. VSCode에서 Live Server로 실행
3. 3개 탭으로 구성:
	- **🔵 버블정렬 시뮬** — 다음 버튼으로 단계별 진행
	- **🎚️ Range 조정** — Range 올리기/낮추기로 색깔 겹침 확인
	- **🔍 Arduino 필터** — 필터 ON/OFF로 노이즈 제거 확인

---

## 📄 JSX 코드

```jsx
import { useState } from "react";

const SIG = {
  1: { name: "빨강", hex: "#ef4444" },
  4: { name: "초록", hex: "#22c55e" },
  5: { name: "파랑", hex: "#3b82f6" },
  6: { name: "보라", hex: "#a855f7" },
};

const RAW = [
  { sig: 4, x: 85, y: 110, w: 38, h: 36 },
  { sig: 1, x: 210, y: 108, w: 35, h: 34 },
  { sig: 6, x: 142, y: 112, w: 37, h: 35 },
  { sig: 5, x: 55, y: 109, w: 36, h: 36 },
];

function buildSortSteps(raw) {
  const steps = [];
  const arr = raw.map(b => ({ ...b }));
  steps.push({ arr: arr.map(b => ({ ...b })), swap: null, msg: "카메라 감지 직후 — X순서 뒤죽박죽", done: false });
  for (let i = 0; i < arr.length - 1; i++) {
    for (let j = 0; j < arr.length - 1 - i; j++) {
      if (arr[j].x > arr[j + 1].x) {
        const t = { ...arr[j] }; arr[j] = { ...arr[j + 1] }; arr[j + 1] = t;
        steps.push({ arr: arr.map(b => ({ ...b })), swap: [j, j + 1], msg: `x[${j}] > x[${j+1}] → 교환!`, done: false });
      } else {
        steps.push({ arr: arr.map(b => ({ ...b })), swap: null, msg: `x[${j}] <= x[${j+1}] → 그대로`, done: false });
      }
    }
  }
  steps.push({ arr: arr.map(b => ({ ...b })), swap: null, msg: "✅ 정렬 완료! Pixy_List[] 저장됨", done: true });
  return steps;
}

const sortSteps = buildSortSteps(RAW);
const RANGE_STEPS = [
  { range: 8.0, status: "good", msg: "감지 잘 됨" },
  { range: 6.0, status: "good", msg: "양호" },
  { range: 4.0, status: "good", msg: "적당함" },
  { range: 2.5, status: "warn", msg: "⚠️ 빨강↔초록 겹치기 시작" },
  { range: 1.5, status: "bad",  msg: "❌ 색깔 구분 안 됨" },
];

export default function App() {
  const [tab, setTab] = useState(0);
  const [step, setStep] = useState(0);
  const [rangeIdx, setRangeIdx] = useState(0);
  const [filterOn, setFilterOn] = useState(false);
  const cur = sortSteps[Math.min(step, sortSteps.length - 1)];
  const rs = RANGE_STEPS[rangeIdx];

  return (
    <div style={{ minHeight: "100vh", background: "#07090f", color: "#c8d6e5",
      fontFamily: "'Noto Sans KR', monospace, sans-serif", padding: "24px 16px 60px" }}>
      <div style={{ textAlign: "center", marginBottom: 24 }}>
        <h1 style={{ fontSize: 20, fontWeight: 900, color: "#fff" }}>
          블록 식별 & 정렬 <span style={{ color: "#00e5ff" }}>시뮬레이터</span>
        </h1>
      </div>
      <div style={{ display: "flex", maxWidth: 720, margin: "0 auto 20px",
        border: "1px solid #1e2d3d", borderRadius: 8, overflow: "hidden" }}>
        {["🔵 버블정렬 시뮬", "🎚️ Range 조정", "🔍 Arduino 필터"].map((t, i) => (
          <button key={i} onClick={() => setTab(i)} style={{
            flex: 1, padding: "10px 6px",
            background: tab === i ? "#0d2137" : "#0d1520", border: "none",
            borderRight: i < 2 ? "1px solid #1e2d3d" : "none",
            color: tab === i ? "#00e5ff" : "#475569",
            fontWeight: tab === i ? 700 : 400, fontSize: 12, cursor: "pointer" }}>{t}</button>
        ))}
      </div>
      {/* 본문은 원본 코드 참조 (3탭: 버블정렬 / Range / 필터) */}
    </div>
  );
}
```

> 💡 본문 JSX 본체(3탭 렌더링 부분)는 원본 Notion 페이지의 코드 블록을 참고해 동일하게 옮길 수 있습니다.
