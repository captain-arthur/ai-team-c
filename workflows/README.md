# Workflows

## 개념

Workflows는 AI 팀이 작업을 수행하는 체계적인 흐름입니다.

표준화된 workflow를 통해 일관된 품질과 효율적인 진행을 보장합니다.

## 기본 Workflow

```
문제 정의 → 연구 → 설계 → 구현 → 검토 → 문서화
   │         │       │       │       │       │
   ▼         ▼       ▼       ▼       ▼       ▼
problem  research  arch   impl   review   docs
```

---

## 실행 모델

### 1. 새 프로젝트 시작

새로운 작업 요청을 받으면:

```
1. manager 역할 활성화
2. project-intake skill 실행
3. projects/_template/ 복사하여 프로젝트 생성
4. 01-problem.md 작성
5. workflow 유형 결정
```

**프로젝트 명명**: `{YYYY-MM}-{project-name}`

### 2. 역할 선택 기준

| 작업 상황 | 활성화할 역할 | 사용할 skill |
|-----------|---------------|--------------|
| 새 작업 수신 | manager | project-intake |
| 정보 부족 | researcher | research-workflow |
| 설계 필요 | architect | architecture-design |
| 구현 필요 | engineer | - |
| 검토 필요 | reviewer | review-checklist |
| 문서화 필요 | writer | - |
| 프로젝트 완료 | manager | project-closeout |

### 3. 산출물 흐름

각 단계의 산출물이 다음 단계의 입력이 됩니다:

```
01-problem.md
     ↓
     ├─→ 조사 필요? → 02-research.md
     │                      ↓
     └─→ 설계 필요? → 03-architecture.md
                            ↓
                      04-engineering/
                            ↓
                      05-review.md
                            ↓
                      06-documentation.md
```

### 4. 지식 추출

프로젝트 완료 시:

```
1. project-closeout skill 실행
2. 재사용 가능한 지식 식별
3. knowledge/ 해당 디렉토리에 저장:
   - design-principles/  ← 설계 원칙
   - patterns/           ← 반복 패턴
   - lessons-learned/    ← 교훈
   - domain-knowledge/   ← 도메인 지식
   - references/         ← 참조 자료
4. 프로젝트 README 업데이트
```

---

## Workflow 유형

### Full Workflow

모든 단계를 거치는 완전한 흐름

**적용**: 복잡한 프로젝트, 중요한 의사결정

```
manager → researcher → architect → engineer → reviewer → writer → manager
   ↓          ↓           ↓           ↓          ↓         ↓        ↓
problem  research    architecture  impl     review    docs   closeout
```

### Research Workflow

연구 중심의 흐름

**적용**: 기술 조사, 옵션 분석

```
manager → researcher → writer → manager
   ↓          ↓          ↓        ↓
problem  research     docs   closeout
```

### Implementation Workflow

구현 중심의 흐름

**적용**: 명확한 요구사항이 있는 개발 작업

```
manager → engineer → reviewer → writer → manager
   ↓         ↓          ↓         ↓        ↓
problem   impl      review     docs   closeout
```

### Quick Fix Workflow

빠른 수정을 위한 흐름

**적용**: 버그 수정, 작은 개선

```
manager → engineer → reviewer
   ↓         ↓          ↓
problem   impl      review
```

---

## Workflow 선택

```
┌─────────────────────────────────────┐
│          작업 요청 수신              │
└─────────────────┬───────────────────┘
                  ▼
         ┌───────────────┐
         │  복잡도 판단   │
         └───────┬───────┘
                 │
    ┌────────────┼────────────┬─────────────┐
    ▼            ▼            ▼             ▼
 복잡함      조사 필요     요구사항 명확   단순 수정
    │            │            │             │
    ▼            ▼            ▼             ▼
  Full       Research    Implementation  Quick Fix
```

---

## 역할과 Workflow

| 단계 | 주요 역할 | 지원 역할 | 산출물 |
|------|----------|----------|--------|
| 문제 정의 | manager | - | 01-problem.md |
| 연구 | researcher | - | 02-research.md |
| 설계 | architect | researcher | 03-architecture.md |
| 구현 | engineer | architect | 04-engineering/ |
| 검토 | reviewer | engineer | 05-review.md |
| 문서화 | writer | reviewer | 06-documentation.md |

---

## Workflow 실행 원칙

### 단계 전환 조건

- 현재 단계의 산출물이 완료됨
- 다음 단계 진행에 필요한 정보가 충분함
- 필요시 이전 단계로 돌아갈 수 있음

### 기록

- 각 단계의 결정사항 기록
- 단계 전환 시 이유 명시
- 최종 산출물 저장

### 유연성

- Workflow는 가이드라인이지 강제 사항 아님
- 상황에 따라 단계 생략 또는 반복 가능
- 중요한 것은 체계적 접근

---

## 빠른 참조

### 새 프로젝트 시작 명령

```
새 프로젝트를 시작합니다: {프로젝트 설명}
```

### 역할 전환 명령

```
{역할} 역할로 {작업}을 수행합니다.
```

### 지식 저장

```
이 내용을 knowledge/{category}/에 저장합니다.
```
