# Engineering: Operational Confidence Framework

**Role:** engineer
**Date:** 2026-03-17

이 디렉토리는 Operational Confidence Framework를 **실제 운영에서 사용할 수 있도록** 구체화한 가이드를 포함합니다.

---

## 구현 내용

1. **signal-specification.md** — 각 신호의 상세 정의 및 PromQL
2. **operational-runbook.md** — 상황별 운영 대응 가이드

---

## 파일 구조

```
04-engineering/
├── README.md (이 파일)
├── signal-specification.md
└── operational-runbook.md
```

---

## 사용 방법

### 일상 운영

1. 대시보드에서 **체인의 오른쪽(Impact, Instability)**을 먼저 확인 → "지금 안전한가?"
2. **체인의 왼쪽(Pressure, Strain)**을 확인 → "내일도 안전한가?"
3. 이상이 있으면 **해당 Stage의 행동 가이드**를 따름

### 대시보드 구축

1. `signal-specification.md`의 PromQL을 사용하여 패널 구성
2. 체인 순서대로 배치: Pressure → Strain → Instability → Impact
3. 각 신호에 threshold 적용하여 색상 코딩

---

## 다음 단계

- [ ] 검토 요청
- [ ] 문서화
