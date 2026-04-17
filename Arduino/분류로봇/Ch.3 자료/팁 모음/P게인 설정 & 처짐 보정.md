---
title: "💡 P게인 설정 & 처짐 보정"
parent: "[[팁 모음]]"
source: "Notion"
tags: [arduino, robot, tips, pgain, sag]
---

# 💡 P게인 설정 & 처짐 보정

## 팔이 아래로 처지는 원인

```
목표 위치 도달 → 모터가 토크로 버팀
시간 경과 → 팔 무게 > 모터 유지력
→ 아래로 밀림

P게인이 높을수록
→ 오차에 더 강하게 반응
→ 중력으로 밀린만큼 더 세게 버팀
→ 처짐이 줄어듦
```

---

## Step 1 — 현재 P게인 확인

`ReadPGain.ino` 업로드 후 시리얼 모니터 확인.

```
예시 출력:
====== 현재 PID 게인 값 ======
         P게인   I게인   D게인
  모터1(ID5):  640     0     0
  모터2(ID6):  640     0     0
  모터3(ID7):  640     0     0
  모터4(ID8):  640     0     0
```

> 기본값: 640

---

## Step 2 — Manipulator.cpp 수정

`InitManipulator()` 함수 안 **토크 ON 바로 위에** 한 줄 추가.

```cpp
for (int i = 0; i < ARM_DXL_ID_CNT; i++) {
  dxl.writeControlTableItem(TORQUE_ENABLE, ARM_DXL_IDS[i], TORQUE_OFF);
  // ... 드라이브 모드, 오퍼레이팅 모드 설정 ...

  // ★ 이 한 줄 추가 (TORQUE_ON 바로 위)
  dxl.writeControlTableItem(POSITION_P_GAIN, ARM_DXL_IDS[i], 800);

  dxl.writeControlTableItem(TORQUE_ENABLE, ARM_DXL_IDS[i], TORQUE_ON);
}
```

---

## Step 3 — 단계별 테스트

**한 번에 확 올리면 진동 위험** — 조금씩 올려가.

| 단계 | P게인 | 상태 |
|------|-------|------|
| 기본 | 640 | 거의 모든 모델 기본값 |
| 1단계 | 720 | 먼저 테스트 |
| 2단계 | 800 | 권장값 — 여기서 멈춰도 충분 |
| 3단계 | 900 | 필요하면 추가 설정 |
| 과도 | 1200+ | ❌ 진동 발생 가능성 높음 |

---

## Step 4 — SagMeasure.ino로 처짐량 확인

```
run,5,1500     ← GRIP_UPPER 실행
wait,10        ← 10초 대기
diff,5         ← 저장값 vs 현재값 비교

✅ 차이값 ±10 이내  → P게인 800 유지
⚠️ 차이값 ±50 이상  → 900으로 올리고 재테스트
❌ 진동 발생       → 다시 640~720으로 낮춰
```

---

## 진동 증상 확인법

```
탬탬탬 소리나 배럴링 떨림
  → P게인 너무 높음

목표를 지나쳤다가 돌아오는 현상
  → D게인을 높이면 완화

리셋 후에도 도달이 느림
  → P게인이 적절한 수준. 유지
```

---

## 모터별 개별 설정 (필요시)

처짐이 특정 모터에서만 심하면 그 모터만 높여도 돼.

```cpp
// 모터별 개별 설정 예시
dxl.writeControlTableItem(POSITION_P_GAIN, ARM_DXL_ID_1, 640); // 1번: 유지
dxl.writeControlTableItem(POSITION_P_GAIN, ARM_DXL_ID_2, 800); // 2번: 올림
dxl.writeControlTableItem(POSITION_P_GAIN, ARM_DXL_ID_3, 800); // 3번: 올림
dxl.writeControlTableItem(POSITION_P_GAIN, ARM_DXL_ID_4, 720); // 4번: 조금
```

> 포인트: 팔을 뻗는 방향 모터(2번, 3번)가 중력 영향을 가장 많이 받아.

---

## 정리

```
첫 시도: 800
  ├─ 처짐 해결 → 확정
  ├─ 진동 발생 → 720으로 낮춰
  └─ 처짐 여전함 → 900으로 올려서 재테스트
```
