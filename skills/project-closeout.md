# project-closeout

## 목적

완료된 프로젝트를 정리하고 학습 내용을 추출합니다.

## 사용 시점

- 프로젝트 목표 달성 후
- 프로젝트 중단 결정 시
- 주요 마일스톤 완료 시

## 입력

- 프로젝트 전체 산출물
- 원래 목표 (`01-problem.md`)

## 절차

### 1. 완료 확인

```markdown
## 목표 달성 확인
- [ ] 목표 1: {상태}
- [ ] 목표 2: {상태}

## 산출물 확인
- [ ] 01-problem.md
- [ ] 02-research.md (해당 시)
- [ ] 03-architecture.md (해당 시)
- [ ] 04-engineering/ (해당 시)
- [ ] 05-review.md
- [ ] 06-documentation.md
```

### 2. 회고

다음 질문에 답합니다:

```markdown
## 잘된 점
- ...

## 개선할 점
- ...

## 예상과 다른 점
- ...

## 다음에 다르게 할 것
- ...
```

### 3. 지식 추출

프로젝트에서 배운 재사용 가능한 지식 식별:

| 지식 | 유형 | 저장 위치 |
|------|------|-----------|
| {내용} | pattern | knowledge/patterns/ |
| {내용} | lesson | knowledge/lessons-learned/ |
| {내용} | principle | knowledge/design-principles/ |

### 4. 기술 추출

반복 사용 가능한 절차 식별:

- 새로운 skill로 정의할 절차가 있는가?
- 기존 skill을 개선할 내용이 있는가?

### 5. 프로젝트 README 업데이트

```markdown
# {프로젝트 이름}

## 상태
[x] 완료

## 목표
{원래 목표}

## 결과
{달성한 것}

## 산출물
- {산출물 1}: {설명}
- {산출물 2}: {설명}

## 학습
- {배운 점 1}
- {배운 점 2}

## 관련 지식
- knowledge/patterns/{관련 패턴}
- knowledge/lessons-learned/{관련 교훈}
```

### 6. 지식 문서 작성

식별된 각 지식에 대해:

```markdown
# {제목}

## 요약
한두 문장 요약

## 상세
자세한 설명

## 적용 상황
이 지식이 유용한 상황

## 출처
projects/{project-name}
```

## 출력

- 업데이트된 프로젝트 README
- 추출된 지식 문서들
- 회고 기록

## 출력 위치

```
projects/{project}/README.md (업데이트)
knowledge/{category}/{topic}.md (새로 생성)
```

## 체크리스트

```markdown
## 프로젝트 종료 체크리스트
- [ ] 모든 목표 확인됨
- [ ] 산출물 정리됨
- [ ] 회고 완료됨
- [ ] 재사용 가능한 지식 추출됨
- [ ] 프로젝트 README 업데이트됨
- [ ] 프로젝트 상태 "완료"로 변경됨
```
