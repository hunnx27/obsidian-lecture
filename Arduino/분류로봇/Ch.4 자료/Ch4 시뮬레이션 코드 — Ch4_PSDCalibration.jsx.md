---
title: "💻 Ch4 시뮬레이션 코드 — Ch4_PSDCalibration.jsx"
parent: "[[Ch.4 — PSD 거리값 보정]]"
source: "Notion"
tags: [arduino, robot, simulation, jsx, psd]
---

# 💻 Ch4 시뮬레이션 코드 — Ch4_PSDCalibration.jsx

## 📌 실행 방법

1. 아래 코드를 `Ch4_PSDCalibration.jsx` 파일로 저장
2. VSCode Live Server로 실행
3. +/- 버튼으로 슬롯 거리값 조정 후 이동 확인
4. 오른쪽 코드 상자에서 **실측값이 자동 생성**됨

---

## 📄 JSX 코드

```jsx
import { useState } from "react";

const SLOTS_REF = [17.5, 24.5, 31.0, 35.5];
const LABELS = ["SLOT 1","SLOT 2","SLOT 3","SLOT 4"];
const COLORS = ["#00e5ff","#22c55e","#fbbf24","#a855f7"];

export default function App() {
  const [slotVals, setSlotVals] = useState([17.5, 24.5, 31.0, 35.5]);
  const [robotPos, setRobotPos] = useState(0);
  const [moving, setMoving] = useState(false);
  const [targetSlot, setTargetSlot] = useState(null);
  const [log, setLog] = useState([]);
  const maxDist = 40;

  const moveTo = (idx) => {
    if (moving) return;
    setMoving(true); setTargetSlot(idx);
    setLog(prev=>[...prev.slice(-4), `슬롯${idx+1}(${slotVals[idx]}cm)으로 이동 중...`]);
    setTimeout(() => {
      setRobotPos(slotVals[idx]); setMoving(false);
      setLog(prev=>[...prev.slice(-4), `✅ 슬롯${idx+1} 도착! PSD = ${slotVals[idx].toFixed(1)}cm`]);
    }, 800);
  };

  const handleSlotChange = (idx, val) => {
    const newVals = [...slotVals]; newVals[idx] = parseFloat(val); setSlotVals(newVals);
  };

  return (
    <div style={{minHeight:"100vh",background:"#07090f",color:"#c8d6e5",
      fontFamily:"'Noto Sans KR',monospace,sans-serif",padding:"24px 16px 60px"}}>
      <div style={{textAlign:"center",marginBottom:24}}>
        <h1 style={{fontSize:20,fontWeight:900,color:"#fff"}}>
          슬롯 거리 <span style={{color:"#00e5ff"}}>보정 시뮬레이터</span></h1>
      </div>

      {/* 시각화 + 슬롯 컨트롤 + 로그 + 자동 코드 생성 */}
      {/* 본문 렌더링은 원본 페이지 참고 */}

      {/* 코드 자동 생성 */}
      <pre>
{`// 현장에서 실측값으로 수정!
float SLOT_PSD[4] = {
  ${slotVals[0].toFixed(1)},  // 슬롯1
  ${slotVals[1].toFixed(1)},  // 슬롯2
  ${slotVals[2].toFixed(1)},  // 슬롯3
  ${slotVals[3].toFixed(1)}   // 슬롯4
};`}
      </pre>
    </div>
  );
}
```

> 💡 시각화/슬롯 컨트롤/로그 패널 등의 본문 렌더링 부분은 원본 Notion 페이지 참고. 위 핵심 로직만으로 동일한 시뮬레이터를 구성할 수 있습니다.
