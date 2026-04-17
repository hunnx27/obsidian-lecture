---
title: "💻 Ch1 시뮬레이션 코드 — Ch1_MissionStructure.jsx"
parent: "[[Ch.1 — 전체 미션 구조 이해]]"
source: "Notion"
tags: [arduino, robot, simulation, jsx]
---

# 💻 Ch1 시뮬레이션 코드 — Ch1_MissionStructure.jsx

## 📌 실행 방법

1. 아래 코드를 `Ch1_MissionStructure.jsx` 파일로 저장
2. VSCode에서 Live Server로 실행
3. 각 단계 클릭 → 오른쪽에서 실행 코드 & 변경 변수 확인

---

## 🎮 주요 기능

- 6단계 플로우차트 클릭형
- 각 단계 클릭 시 실행 코드 표시
- 해당 단계에서 변경되는 변수 실시간 표시
- 핵심 루프(슬롯 순회) 요약 패널

---

## 📄 JSX 코드

```jsx
import { useState } from "react";

const steps = [
  {
    id: 1, icon: "⚙️", label: "초기화 & 버튼 대기",
    color: "#00e5ff",
    desc: "모터·센서·카메라 초기화. 집게 열고 INITIAL 자세. SW1 버튼 누를 때까지 대기.",
    code: `InitManipulator(dxl);
OpenGripper(pixy);
RunManipulatorPoseWithPoseDataInEEPROM(dxl, INITIAL, 1000);
while (digitalRead(SW1_PIN) != 0) {} // 버튼 대기`,
    vars: { pose: "INITIAL(1)", gripperState: "OPEN", lampState: "OFF" }
  },
  {
    id: 2, icon: "🚗", label: "장애물 접근 & 미션존 이동",
    color: "#fbbf24",
    desc: "PID 제어로 7.5cm까지 접근 후 11.5cm로 후진 정렬. 카메라 램프 ON. 왼쪽으로 슬라이딩.",
    code: `PID_AlignAndApproach(dxl, OBS=7.5, ...);
PID_AlignAndApproach(dxl, OBSB=11.5, ...);
RunManipulatorPoseWithPoseDataInEEPROM(dxl, MISSION_1, 1000);
pixy.setLamp(1, 1);
DriveLeftWithSideLeftSensorPID(dxl, MDZ=14, ...);`,
    vars: { pose: "MISSION_1(2)", lampState: "ON" }
  },
  {
    id: 3, icon: "📷", label: "블록 전체 스캔 (순서 결정)",
    color: "#00e5ff",
    desc: "Pixy2로 블록 한 번에 촬영. X좌표 기준 버블정렬로 왼→오른 순서 확정. Pixy_List[]에 저장.",
    code: `Pixy_xNUM_OneShot();
// 내부 동작:
// 1. pixy.ccc.getBlocks()
// 2. X좌표 기준 버블정렬
// 결과: Pixy_List[] = [sig5, sig4, sig6, sig1]`,
    vars: { Pixy_B: 4, "Pixy_List[]": "[5,4,6,1]", findBoxNum: 4 }
  },
  {
    id: 4, icon: "🎯", label: "적재함 앞 정밀 정렬",
    color: "#fbbf24",
    desc: "SEE_STORAGE 자세 전환. 40cm PID 접근. 앞+옆 PSD 동시 사용해 직각 2축 정밀 정렬.",
    code: `RunManipulatorPoseWithPoseDataInEEPROM(dxl, SEE_STORAGE, 1000);
Forward(dxl, 200);
PID_Approach(dxl, 40, ...);
AlignUsingFrontAndSide430PID(dxl, FP=43, SP=6, ...);`,
    vars: { pose: "SEE_STORAGE(4)" }
  },
  {
    id: 5, icon: "🔄", label: "슬롯 순회 → 집기 → 배치 [핵심 루프]",
    color: "#39ff14",
    desc: "남은 블록이 있는 동안 반복. 슬롯1→4 탐색 후 목표 색깔 발견 시 집어서 지정 위치에 배치.",
    code: `while (findBoxNum > 0) {
  int targetColor = Pixy_List[count];
  for (int slot=0; slot<=NUM_SLOTS; slot++) {
    DriveLeftWithSideLeftSensorPID(dxl, SLOT_PSD[slot], ...);
    pixy.ccc.getBlocks();
    if (블록 발견) {
      Grap_blocks(y);
      PUT_ON(Put_Position_ByColor[targetColor]);
      count++; findBoxNum--;
      break;
    }
  }
}`,
    vars: { count: "0→1→2→3", findBoxNum: "4→3→2→1→0" }
  },
  {
    id: 6, icon: "🏠", label: "복귀",
    color: "#00e5ff",
    desc: "카메라 램프 OFF. PSD로 구역 경계 감지하며 후진. INITIAL 자세 복귀 후 왼쪽 이동 → 정지.",
    code: `pixy.setLamp(0, 0);
while(SL_PSD_Save() < 15) Backward(dxl, 250);
RunManipulatorPoseWithPoseDataInEEPROM(dxl, INITIAL, 1000);
Left(dxl, 150);
Stop(dxl, 0);`,
    vars: { pose: "INITIAL(1)", lampState: "OFF", gripperState: "OPEN" }
  }
];

export default function App() {
  const [active, setActive] = useState(null);
  const cur = active !== null ? steps[active] : null;

  return (
    <div style={{ minHeight: "100vh", background: "#07090f", color: "#c8d6e5", fontFamily: "'Noto Sans KR', monospace, sans-serif", padding: "24px 16px 60px" }}>
      <div style={{ textAlign: "center", marginBottom: 28 }}>
        <div style={{ display: "inline-block", fontFamily: "monospace", fontSize: 10, letterSpacing: 3, color: "#00e5ff", border: "1px solid #00e5ff33", padding: "3px 12px", marginBottom: 10 }}>Ch.1 — 전체 미션 구조</div>
        <h1 style={{ fontSize: 22, fontWeight: 900, color: "#fff" }}>색깔 블록 분류 로봇<br /><span style={{ color: "#00e5ff" }}>전체 동작 흐름</span></h1>
        <p style={{ fontSize: 11, color: "#334155", fontFamily: "monospace", marginTop: 6 }}>단계 클릭 → 코드 & 변수 확인</p>
      </div>
      <div style={{ display: "flex", gap: 16, maxWidth: 860, margin: "0 auto", alignItems: "flex-start" }}>
        <div style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center" }}>
          {steps.map((s, i) => (
            <div key={s.id} style={{ width: "100%", display: "flex", flexDirection: "column", alignItems: "center" }}>
              <div onClick={() => setActive(active === i ? null : i)} style={{ width: "100%", padding: "12px 16px", background: active === i ? s.color + "22" : "#0f1520", border: `${active === i ? 2 : 1}px solid ${active === i ? s.color : s.color + "44"}`, borderRadius: 6, cursor: "pointer", transition: "all 0.2s", display: "flex", alignItems: "center", gap: 12 }}>
                <span style={{ fontSize: 20 }}>{s.icon}</span>
                <div style={{ flex: 1 }}>
                  <div style={{ fontSize: 13, fontWeight: 700, color: active === i ? s.color : "#e2e8f0" }}>{s.label}</div>
                  {active === i && <div style={{ fontSize: 11, color: "#64748b", marginTop: 3 }}>{s.desc}</div>}
                </div>
                <div style={{ fontFamily: "monospace", fontSize: 10, color: s.color, background: s.color + "11", border: `1px solid ${s.color}33`, padding: "2px 8px", borderRadius: 10 }}>0{s.id}</div>
              </div>
              {i < steps.length - 1 && <div style={{ display: "flex", flexDirection: "column", alignItems: "center", margin: "2px 0" }}><div style={{ width: 1, height: 14, background: "#1e2d3d" }} /><div style={{ color: "#334155", fontSize: 10 }}>▼</div></div>}
            </div>
          ))}
        </div>
        <div style={{ width: 260, flexShrink: 0, display: "flex", flexDirection: "column", gap: 10 }}>
          <div style={{ background: "#0d1520", border: "1px solid #1e2d3d", borderRadius: 8, padding: 14, minHeight: 120 }}>
            <div style={{ fontFamily: "monospace", fontSize: 9, letterSpacing: 2, color: "#475569", marginBottom: 8, textTransform: "uppercase" }}>📄 실행 코드</div>
            {cur ? <pre style={{ fontFamily: "monospace", fontSize: 11, color: "#7dd3fc", lineHeight: 1.7, margin: 0, whiteSpace: "pre-wrap" }}>{cur.code}</pre> : <div style={{ color: "#1e2d3d", fontSize: 11, textAlign: "center", paddingTop: 30 }}>← 단계를 클릭하세요</div>}
          </div>
          <div style={{ background: "#0d1520", border: "1px solid #1e2d3d", borderRadius: 8, padding: 14 }}>
            <div style={{ fontFamily: "monospace", fontSize: 9, letterSpacing: 2, color: "#475569", marginBottom: 8, textTransform: "uppercase" }}>📟 변경되는 변수</div>
            {cur ? Object.entries(cur.vars).map(([k, v]) => (<div key={k} style={{ display: "flex", justifyContent: "space-between", padding: "5px 8px", background: cur.color + "11", border: `1px solid ${cur.color}33`, borderRadius: 4, marginBottom: 4 }}><span style={{ fontFamily: "monospace", fontSize: 10, color: "#94a3b8" }}>{k}</span><span style={{ fontFamily: "monospace", fontSize: 11, fontWeight: 700, color: "#e2e8f0" }}>{String(v)}</span></div>)) : <div style={{ color: "#1e2d3d", fontSize: 11, textAlign: "center", paddingTop: 10 }}>없음</div>}
          </div>
        </div>
      </div>
    </div>
  );
}
```
