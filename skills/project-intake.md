# project-intake

## 목적

새로운 작업 요청을 체계적으로 분석하고 프로젝트로 구조화합니다.

## 사용 시점

- 사용자가 새로운 작업을 요청할 때
- 복잡한 문제를 해결해야 할 때
- 명확한 범위 정의가 필요할 때

## 입력

- 사용자의 요청 또는 문제 설명
- 관련 컨텍스트 (있는 경우)

## 절차

### 1. 요청 분석

다음 질문에 답합니다:

- 무엇을 달성하려 하는가?
- 왜 이것이 필요한가?
- 누가 결과물을 사용하는가?
- 제약 조건은 무엇인가?

### 2. 범위 결정

- 포함되는 것 (in scope)
- 제외되는 것 (out of scope)
- 의존성 및 전제 조건

### 3. 작업 분류

작업 유형 판단:

| 유형 | 설명 | workflow |
|------|------|----------|
| research | 정보 수집 및 분석 | research workflow |
| design | 시스템/솔루션 설계 | full workflow |
| implementation | 코드/구성 작성 | implementation workflow |
| fix | 문제 수정 | quick fix workflow |
| documentation | 문서 작성 | documentation workflow |

### 4. 프로젝트 생성

```bash
# 프로젝트 디렉토리 생성
projects/{YYYY-MM}-{project-name}/
```

`projects/_template/`를 복사하여 시작합니다.

### 5. 문제 정의 작성

`01-problem.md`에 다음을 기록:

- 목표
- 배경
- 범위
- 성공 기준
- 제약 조건

## 출력

- 프로젝트 디렉토리
- 작성된 `01-problem.md`
- 선택된 workflow

## 출력 위치

```
projects/{YYYY-MM}-{project-name}/
├── README.md
└── 01-problem.md
```

## 다음 단계

- research가 필요하면 → `research-workflow` 실행
- 설계가 필요하면 → `architecture-design` 실행
- 바로 구현 가능하면 → engineer 역할 활성화
