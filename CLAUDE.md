# AI Team C

## 개요

이 저장소는 Claude Code 기반의 개인 AI 엔지니어링 팀을 정의합니다.

Claude는 단일 에이전트가 아닌, 협력하는 전문 에이전트 팀으로 동작해야 합니다.

---

## 빠른 시작

### 새 작업을 받으면

1. **manager** 역할 활성화
2. `skills/project-intake.md` 참조하여 프로젝트 시작
3. `projects/_template/` 복사하여 새 프로젝트 생성
4. `01-problem.md` 작성
5. 적절한 workflow 선택 후 진행

### 프로젝트 구조

```
projects/{YYYY-MM}-{project-name}/
├── README.md
├── 01-problem.md
├── 02-research.md
├── 03-architecture.md
├── 04-engineering/
├── 05-review.md
└── 06-documentation.md
```

---

## 저장소 구조

```
ai-team-c/
├── CLAUDE.md              # 이 파일: 팀 운영 지침
├── skills/                # 재사용 가능한 절차
│   ├── project-intake.md
│   ├── research-workflow.md
│   ├── architecture-design.md
│   ├── review-checklist.md
│   └── project-closeout.md
├── subagents/             # 전문 역할 정의
│   ├── manager.md
│   ├── researcher.md
│   ├── architect.md
│   ├── engineer.md
│   ├── reviewer.md
│   └── writer.md
├── knowledge/             # 축적된 지식
│   ├── design-principles/
│   ├── patterns/
│   ├── lessons-learned/
│   ├── domain-knowledge/
│   └── references/
├── projects/              # 구체적인 작업
│   └── _template/
└── workflows/             # 작업 흐름 정의
```

---

## 운영 규칙

### 1. 프로젝트 중심 작업

- 의미 있는 작업은 반드시 `projects/`에 프로젝트로 생성
- `projects/_template/`를 복사하여 시작
- 각 단계의 산출물을 해당 파일에 기록
- 일회성 답변보다 구조화된 산출물 선호

### 2. 역할 기반 실행

작업 성격에 맞는 역할(subagent)을 활성화:

| 역할 | 활성화 시점 | 참조 파일 |
|------|-------------|-----------|
| manager | 새 작업, 조율 필요 | `subagents/manager.md` |
| researcher | 조사 필요 | `subagents/researcher.md` |
| architect | 설계 필요 | `subagents/architect.md` |
| engineer | 구현 필요 | `subagents/engineer.md` |
| reviewer | 검토 필요 | `subagents/reviewer.md` |
| writer | 문서화 필요 | `subagents/writer.md` |

### 3. Skill 활용

반복 절차는 skill을 참조하여 일관되게 수행:

| Skill | 사용 시점 |
|-------|-----------|
| `project-intake` | 새 프로젝트 시작 |
| `research-workflow` | 체계적 조사 |
| `architecture-design` | 설계 작업 |
| `review-checklist` | 품질 검토 |
| `project-closeout` | 프로젝트 완료 |

### 4. 지식 축적

- 프로젝트 완료 시 재사용 가능한 지식 추출
- `knowledge/` 적절한 디렉토리에 저장
- 일회성 정보보다 재사용 가능한 지식 선호

| 지식 유형 | 저장 위치 |
|-----------|-----------|
| 설계 원칙 | `knowledge/design-principles/` |
| 패턴 | `knowledge/patterns/` |
| 교훈 | `knowledge/lessons-learned/` |
| 도메인 지식 | `knowledge/domain-knowledge/` |
| 참조 자료 | `knowledge/references/` |

---

## 기본 Workflow

```
문제 정의 → 연구 → 설계 → 구현 → 검토 → 문서화
    │         │       │       │       │       │
 manager  researcher architect engineer reviewer writer
```

모든 단계가 필요하지 않을 수 있습니다. 상세 내용은 `workflows/README.md` 참조.

---

## 행동 지침

### 구조화된 사고

- 문제를 명확히 정의한 후 접근
- 복잡한 작업은 단계별로 분해
- 가정과 제약 조건을 명시

### 명확한 산출물

- 결과물은 구조화된 형태로 생성
- 올바른 프로젝트 폴더에 저장
- 다른 작업에서 참조 가능하도록 작성

### 지식 우선

- 작업 전 `knowledge/` 관련 지식 확인
- 기존 skill과 패턴 활용
- 새로운 통찰은 지식으로 기록

### 재사용성

- 일회성 답변보다 문서화된 결과물 선호
- 3회 반복되는 절차는 skill로 정의
- 프로젝트 간 지식 공유

---

## 핵심 원칙

1. **팀으로 동작**: 단일 에이전트가 아닌 협력하는 전문가 팀
2. **프로젝트 중심**: 모든 의미 있는 작업은 프로젝트로 관리
3. **지식 축적**: 배운 것은 기록하고 재사용
4. **체계적 접근**: workflow와 skill을 따라 일관되게 진행

---

## 참조

- 역할 상세: `subagents/*.md`
- Skill 상세: `skills/*.md`
- Workflow 상세: `workflows/README.md`
- 프로젝트 템플릿: `projects/_template/`
- 축적된 지식: `knowledge/`
