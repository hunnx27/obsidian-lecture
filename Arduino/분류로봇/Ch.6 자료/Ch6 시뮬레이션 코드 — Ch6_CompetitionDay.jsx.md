---
title: "💻 Ch6 시뮬레이션 코드 — Ch6_CompetitionDay.jsx"
parent: "[[Ch.6 — 당일 체크리스트]]"
source: "Notion"
tags: [arduino, robot, simulation, jsx, checklist]
---

# 💻 Ch6 시뮬레이션 코드 — Ch6_CompetitionDay.jsx

## 📌 실행 방법

1. 아래 코드를 `Ch6_CompetitionDay.jsx` 파일로 저장
2. VSCode Live Server로 실행
3. 3개 탭 활용:
	- **✅ 체크리스트** — 클릭마다 체크 + 진행률 바
	- **🔧 트러블슈팅** — 증상별 원인/해결책 안내
	- **💡 최종 요약** — 3가지 체크 + 당일 순서

---

## 📋 체크리스트 데이터 (요약)

**🌅 경연 당일 아침**
- EEPROM 자세 저장 목록 확인 (`list` 명령어) — 필수
- 모든 자세 `run`으로 실제 동작 확인 — 필수
- 배터리 충전 상태 확인
- 대회 코드로 업로드 완료 확인 — 필수

**📷 카메라 세팅**
- 경연장 조명에서 Pixy2 시그니처 재학습 — 필수
- 4색 블록 모두 정확히 감지되는지 확인 — 필수
- 교차 오염 없는지 확인 (빨강이 초록으로 인식 X) — 필수
- Range 값 현장에서 재조정

**📏 PSD 보정**
- 슬롯1~4 (`SLOT_PSD[0]`~`[3]`) 실측값 수정 — 필수

**🔁 시뮬레이션**
- 전체 1회 완주 테스트 — 필수
- 블록 4개 모두 정확한 위치에 배치 확인 — 필수
- 복귀 동작 정상 확인

---

## 🔧 트러블슈팅 표

| 증상 | 원인 | 해결 |
|------|------|------|
| 팔이 안 움직임 | EEPROM 저장 누락 | `list` → 누락 번호 확인 → 재저장 |
| 블록을 못 찾음 | 카메라 Range 오류 또는 다른 조명 | PixyMon 재학습, Range 높이기 |
| 슬롯 엉뚱한 곳 정지 | SLOT_PSD 실측 안 함 | PSD 실측 코드로 실측 |
| 집는데 놓침 | GRIP_PICK 자세 위치 틀림 | SavePose → run으로 재확인 후 재저장 |
| 색깔 오분류 | 시그니처 학습 문제 | Pixy2 재학습 |

---

## 📄 JSX 코드 (핵심 데이터 구조)

```jsx
import { useState } from "react";

const CHECKLIST = [
  { phase:"🌅 경연 당일 아침", color:"#fbbf24", items:[
    {id:"c1",text:"EEPROM 자세 저장 목록 확인 (list 명령어)",critical:true},
    {id:"c2",text:"모든 자세 run으로 실제 동작 확인",critical:true},
    {id:"c3",text:"배터리 충전 상태 확인",critical:false},
    {id:"c4",text:"대회 코드로 업로드 완료 확인",critical:true},
  ]},
  { phase:"📷 카메라 세팅", color:"#00e5ff", items:[
    {id:"p1",text:"경연장 조명에서 Pixy2 시그니처 재학습",critical:true},
    {id:"p2",text:"4색 블록 모두 정확히 감지되는지 확인",critical:true},
    {id:"p3",text:"교차 오염 없는지 확인",critical:true},
    {id:"p4",text:"Range 값 현장에서 재조정",critical:false},
  ]},
  { phase:"📏 PSD 보정", color:"#22c55e", items:[
    {id:"s1",text:"슬롯1(SLOT_PSD[0]) 실측값 수정",critical:true},
    {id:"s2",text:"슬롯2(SLOT_PSD[1]) 실측값 수정",critical:true},
    {id:"s3",text:"슬롯3(SLOT_PSD[2]) 실측값 수정",critical:true},
    {id:"s4",text:"슬롯4(SLOT_PSD[3]) 실측값 수정",critical:true},
  ]},
  { phase:"🔁 시뮬레이션", color:"#39ff14", items:[
    {id:"r1",text:"전체 1회 완주 테스트",critical:true},
    {id:"r2",text:"블록 4개 모두 정확한 위치에 배치 확인",critical:true},
    {id:"r3",text:"복귀 동작 정상 확인",critical:false},
  ]},
];

const TROUBLES = [
  {symptom:"팔이 안 움직임", cause:"EEPROM 저장 누락", fix:"list → 누락 번호 확인 → 재저장"},
  {symptom:"블록을 못 찾음", cause:"카메라 Range 오류 또는 조명 변화", fix:"PixyMon 재학습, Range 높이기"},
  {symptom:"슬롯 엉뚱한 곳 정지", cause:"SLOT_PSD 실측 안 함", fix:"PSD 실측 코드 업로드 후 실측"},
  {symptom:"집는데 놓침", cause:"GRIP_PICK 자세 위치 틀림", fix:"SavePose → run으로 재확인 후 재저장"},
  {symptom:"색깔 오분류", cause:"시그니처 학습 문제", fix:"Pixy2 재학습"},
];

// ... 본문 렌더링은 원본 페이지 참고 (탭 + 진행률 바 + 카드)
```

---

## 🏆 최종 핵심 3가지

1. **카메라로 순서 파악** — Pixy2 시그니처 학습 + X좌표 정렬 + Pixy_List[] 확정
2. **팔 자세 저장** — SavePose 코드 + 팔 이동 + save 명령 + run 확인
3. **거리값 보정** — PSD 실측 코드 + 슬롯별 측정 + SLOT_PSD[] 수정
