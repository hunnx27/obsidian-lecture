---
title: "💻 Ch5 시뮬레이션 코드 — Ch5_VariableConnection.jsx"
parent: "[[Ch.5 — 핵심 변수 연결 구조]]"
source: "Notion"
tags: [arduino, robot, simulation, jsx]
---

# 💻 Ch5 시뮬레이션 코드 — Ch5_VariableConnection.jsx

## 📌 실행 방법

1. 아래 코드를 `Ch5_VariableConnection.jsx` 파일로 저장
2. VSCode Live Server로 실행
3. 3개 탭 활용:
	- **🔗 연결 흐름** — 색깔 클릭 → Pixy_List→count→targetColor→putPos→EEPROM 전체 추적
	- **🎮 루프 시뮬** — 다음 버튼으로 count++ 파정
	- **⚠️ 실패 케이스** — 도미노 한 개라도 잘못되면?

---

## 📄 JSX 코드

```jsx
import { useState } from "react";

const SIG = {
  1:{name:"빨강",hex:"#ef4444"},
  4:{name:"초록",hex:"#22c55e"},
  5:{name:"파랑",hex:"#3b82f6"},
  6:{name:"보라",hex:"#a855f7"},
};
const PUT_MAP = {1:13, 4:10, 5:15, 6:11};
const EEPROM_POSES = {
  10:"PUT_GREEN (초록 배치)",
  11:"PUT_PURPLE (보라 배치)",
  13:"PUT_RED (빨강 배치)",
  15:"PUT_BLUE (파랑 배치)",
};
const PIXY_LIST = [5, 1, 4, 6];

function InfoBox({label, value, color}) {
  return (
    <div style={{flex:1,background:color+"11",border:`1px solid ${color}33`,
      borderRadius:6,padding:"8px 10px"}}>
      <div style={{fontFamily:"monospace",fontSize:9,color:"#475569"}}>{label}</div>
      <div style={{fontSize:12,fontWeight:700,color:"#e2e8f0",fontFamily:"monospace",marginTop:3}}>{value}</div>
    </div>
  );
}

export default function App() {
  const [selected, setSelected] = useState(null);
  const [count, setCount] = useState(0);
  const [tab, setTab] = useState(0);
  const targetSig = PIXY_LIST[count] ?? null;
  const targetC   = targetSig ? SIG[targetSig] : null;
  const putPos    = targetSig ? PUT_MAP[targetSig] : null;

  // ... (3탭: 연결 흐름 / 루프 시뮬 / 실패 케이스)
  // 본문 렌더링은 원본 페이지 참고
}
```

> 💡 본문은 3탭 렌더링 (연결 흐름 / 루프 시뮬 / 실패 케이스). 전체 본체는 원본 Notion 페이지 참고.

---

## 핵심 데이터 매핑

| Pixy sig | 색깔 | Put_Position_ByColor | EEPROM 자세 |
|----------|------|----------------------|-------------|
| 1 | 빨강 | 13 | PUT_RED |
| 4 | 초록 | 10 | PUT_GREEN |
| 5 | 파랑 | 15 | PUT_BLUE |
| 6 | 보라 | 11 | PUT_PURPLE |

---

## ⚠️ 실패 케이스 요약

| 변수 | 실패 원인 | 결과 | 해결 |
|------|-----------|------|------|
| `Pixy_List[]` | 카메라 시그니처 오학습 | 색깔 순서 틀림 → 엉뚱한 블록 집음 | PixyMon에서 시그니처 재학습 |
| `SLOT_PSD[]` | 실측 없이 기본값 사용 | 슬롯에 안 서고 엉뚱한 위치에 정지 | 당일 실측값으로 수정 |
| `Put_Position_ByColor[]` | 인덱스 불일치 | 색깔별 배치 위치 오류 | sig1→[1], sig4→[4] 값 확인 |
| EEPROM 자세번호 | 저장 안 한 번호 호출 | `isTherePoseData=false` → 팔 안 움직임 | `list`로 누락 확인 후 재저장 |
