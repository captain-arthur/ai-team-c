# Skills

## 개념

Skills는 AI 팀이 반복적으로 수행하는 재사용 가능한 절차입니다.

잘 정의된 skill은:

- 명확한 목적을 가집니다
- 입력과 출력이 정의되어 있습니다
- 단계별 절차가 있습니다
- 여러 프로젝트에서 활용됩니다

## 구조

각 skill은 다음 형태로 정의됩니다:

```
skills/
├── README.md
├── {skill-name}/
│   ├── README.md      # skill 설명
│   ├── prompt.md      # 실행 프롬프트 (선택)
│   └── examples/      # 사용 예시 (선택)
```

## Skill 정의 템플릿

```markdown
# {Skill Name}

## 목적
이 skill이 해결하는 문제

## 입력
- 필요한 정보나 컨텍스트

## 출력
- 생성되는 결과물

## 절차
1. 첫 번째 단계
2. 두 번째 단계
3. ...

## 주의사항
- 고려해야 할 점
```

## 예시 Skills (향후 정의)

| Skill | 설명 |
|-------|------|
| `research` | 주제에 대한 체계적 조사 |
| `architecture-review` | 시스템 설계 검토 |
| `incident-investigation` | 장애 원인 분석 |
| `slo-analysis` | SLO 설계 및 분석 |
| `dashboard-design` | 모니터링 대시보드 설계 |
| `code-review` | 코드 품질 검토 |
| `documentation` | 기술 문서 작성 |

## 사용 방법

1. 필요한 skill이 있는지 `skills/` 확인
2. 있다면 해당 skill의 절차를 따름
3. 없다면 작업 수행 후 반복될 가능성이 있으면 skill로 정의

## Skill 추가 기준

다음 경우 새 skill을 정의합니다:

- 동일한 절차를 3회 이상 반복
- 절차가 복잡하여 표준화가 필요
- 여러 프로젝트에서 활용 가능
