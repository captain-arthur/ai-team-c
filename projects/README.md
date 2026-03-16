# Projects

## 개념

Projects는 AI 팀이 수행하는 구체적인 작업 단위입니다.

모든 의미 있는 작업은 프로젝트로 관리되며, 체계적인 과정을 통해 진행됩니다.

## 프로젝트 구조

```
projects/
├── README.md
├── {project-name}/
│   ├── README.md           # 프로젝트 개요
│   ├── 01-problem.md       # 문제 정의
│   ├── 02-research.md      # 연구 결과
│   ├── 03-architecture.md  # 설계
│   ├── 04-implementation/  # 구현 산출물
│   ├── 05-review.md        # 검토 결과
│   └── 06-documentation.md # 최종 문서
```

모든 단계가 필요하지는 않으며, 프로젝트 성격에 따라 조정합니다.

## 프로젝트 진행 단계

### 1. 문제 정의 (problem)

- 해결하려는 문제 명확화
- 목표와 성공 기준 정의
- 범위와 제약 조건 명시

### 2. 연구 (research)

- 관련 정보 수집
- 기존 해결책 조사
- 옵션 분석

### 3. 설계 (architecture)

- 해결 방안 설계
- 기술 선택
- Trade-off 분석

### 4. 구현 (implementation)

- 실제 구현 작업
- 코드, 설정, 스크립트 등

### 5. 검토 (review)

- 결과물 검토
- 품질 확인
- 개선 사항 식별

### 6. 문서화 (documentation)

- 최종 결과 정리
- 사용 방법 문서화
- 배운 점 기록

## 프로젝트 README 템플릿

```markdown
# {프로젝트 이름}

## 상태
[ ] 진행 중 / [x] 완료 / [ ] 보류

## 목표
달성하려는 것

## 배경
왜 이 프로젝트가 필요한지

## 결과
주요 산출물 요약

## 학습
프로젝트에서 배운 점
```

## 프로젝트 명명 규칙

```
{날짜}-{주제}
```

예시:
- `2024-01-monitoring-dashboard`
- `2024-01-api-optimization`
- `2024-02-incident-analysis`

## 프로젝트 관리 원칙

- 시작 전 문제를 명확히 정의
- 각 단계의 산출물을 기록
- 완료 후 학습 내용을 knowledge/에 추가
- 재사용 가능한 절차는 skills/로 추출
