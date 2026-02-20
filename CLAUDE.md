# accounting-review-team · Claude CoWork 실행 단위 지침서

트리거 조건: 사용자가 파일을 업로드하고 분석을 요청하면, 반드시 아래 워크플로우를 따릅니다.

이 문서는 회계 자료를 분석하고 경영자용 보고서를 생성할 때 따라야 하는 표준 실행 워크플로우를 정의합니다.
입력 파일이 PDF인 경우 반드시 `.claude/skills/pdf-read/SKILL.md` 스킬을 사용하여 읽습니다.

이 지침서는 다음 목적을 가집니다:

- 실행 단위별(run) 결과를 독립적으로 관리
- 토큰 사용량 최소화
- 분석 결과(JSON)와 보고서(인간용 문서)를 명확히 분리
- 에이전트 간 일관된 협업 구조 유지

---

# 프로젝트 구조

본 프로젝트는 아래 구조를 기준으로 동작합니다.

```
accounting-review-team/
├── .claude/
│   ├── agents/                     # 에이전트 역할/프로세스 정의
│   └── skills/                     # 재사용 가능한 스킬 정의/구현
│
├── outputs/                        # 산출물 저장소
│   └── yyyy-mm/                    # 실행 단위 폴더 (예: 2026-03)
│       ├── reference/              # 에이전트 산출물(JSON) 모음
│       ├── yyyy-mm-경영자보고서.pdf
│       └── _manifest.json
│
└── CLAUDE.md                       # 실행 단위 지침서(본 문서)
```

## 폴더별 역할

- `.claude/agents/` : 각 에이전트의 역할과 작업 프로세스만 정의합니다.
- `.claude/skills/` : 반복 작업을 코드/스킬로 캡슐화합니다. 보고서 생성에 필요한 변환(예: docx→pdf) 등을 여기에 둘 수 있습니다.
- `outputs/` : 모든 결과물은 반드시 실행 단위(run_id) 폴더 아래에만 생성합니다.
- `outputs/yyyy-mm/reference/` : 토큰 낭비를 줄이기 위해, 분석 결과는 JSON으로만 저장합니다.
- `outputs/yyyy-mm/*.pdf` : 경영자용 최종 산출물은 report-writer만 생성합니다.

---

# 핵심 원칙

## 1. 실행 단위(run)는 yyyy-mm 기준으로 관리

모든 실행은 반드시 다음 형식의 run_id를 사용합니다:

### run_id 결정 기준 (중요)

run_id는 **문서의 기준 기간(document reporting period)** 을 기준으로 결정해야 합니다.

즉, 실행 시점의 현재 날짜가 아니라, 업로드된 회계 자료에 명시된 기간을 사용합니다.

예:

- 합계잔액시산표 기준일: 2025-09-30 → run_id = 2025-09
- 손익계산서 기간: 2025년 9월 → run_id = 2025-09
- 재무상태표 기준일: 2025-12-31 → run_id = 2025-12

절대 다음 기준을 사용하지 않습니다:

- 실행한 현재 날짜
- 시스템 날짜

예외:

문서에서 기간을 명확히 식별할 수 없는 경우에만 현재 날짜를 사용하여 run_id를 생성할 수 있습니다.

우선순위:

1. 문서에 명시된 기준일 (가장 우선)
2. 문서에 명시된 회계 기간
3. 문서 파일명에 포함된 기간
4. (예외) 현재 날짜


예:

2026-01  
2026-02  
2026-03  

분기 보고서가 필요한 경우에도 해당 분기의 마지막 월 기준으로 사용합니다.

예:

2026년 1분기 → 2026-03  
2026년 2분기 → 2026-06  

---

## 2. 모든 산출물은 run_id 폴더 내부에 저장

모든 파일은 다음 경로를 기준으로 생성합니다:

outputs/{run_id}/

예:

outputs/2026-03/

---

## 3. 폴더 구조 규칙

각 run_id 폴더는 다음 구조를 따릅니다:

outputs/{run_id}/
├── reference/
│   ├── normalized.json
│   ├── completeness.json
│   ├── classification.json
│   ├── anomaly.json
│   ├── tax_risk.json
│   ├── balance.json
├── {run_id}-경영자보고서.pdf
└── _manifest.json

- `reference/` : 에이전트 산출물(JSON) 전용 폴더
- `{run_id}-경영자보고서.*` : report-writer가 생성하는 최종 보고서 파일

---

## 4. JSON 우선 원칙 (Token 최소화)

다음 에이전트의 모든 산출물은 반드시 JSON 형식으로 생성합니다:

- normalizer
- completeness-agent
- classification-agent
- anomaly-agent
- tax-risk-agent
- balance-agent

JSON 사용 목적:

- 토큰 사용량 최소화
- 구조화된 데이터 유지
- report-writer가 효율적으로 분석 가능

절대로 인간용 설명을 포함한 긴 문서를 생성하지 않습니다.

---

## 5. report-writer만 인간용 보고서 생성

다음 에이전트만 인간용 문서를 생성할 수 있습니다:

report-writer

report-writer는 다음 형식의 파일을 생성합니다:

outputs/{run_id}/{run_id}-경영자보고서.pdf

PDF 생성 시 반드시 `.claude/skills/pdf-generate/SKILL.md` 스킬을 사용합니다.

---

## 6. 에이전트별 파일 생성 위치 규칙

각 에이전트는 반드시 다음 위치에만 파일을 생성합니다:

normalizer:
outputs/{run_id}/reference/normalized.json

completeness-agent:
outputs/{run_id}/reference/completeness.json

classification-agent:
outputs/{run_id}/reference/classification.json

anomaly-agent:
outputs/{run_id}/reference/anomaly.json

tax-risk-agent:
outputs/{run_id}/reference/tax_risk.json

balance-agent:
outputs/{run_id}/reference/balance.json

report-writer:
outputs/{run_id}/{run_id}-경영자보고서.md

---

## 7. Manifest 파일 관리 규칙

각 run에는 반드시 manifest 파일이 존재해야 합니다:

outputs/{run_id}/_manifest.json

manifest 목적:

- 실행 상태 추적
- 완료된 에이전트 확인
- report-writer 실행 준비 상태 확인

manifest 예시:

{
  "run_id": "2026-03",
  "created_at": "ISO-8601",
  "status": {
    "normalized": false,
    "completeness": false,
    "classification": false,
    "anomaly": false,
    "tax_risk": false,
    "balance": false,
    "report": false
  }
}

각 에이전트는 작업 완료 후 자신의 상태를 true로 변경해야 합니다.

---

## 8. 실행 순서 규칙

에이전트는 반드시 다음 순서를 따릅니다:

1. normalizer
2. completeness-agent
3. classification-agent
4. anomaly-agent
5. tax-risk-agent
6. balance-agent
7. report-writer

report-writer는 outputs/{run_id}/reference/ 아래에 필요한 JSON이 모두 존재할 때만 실행할 수 있습니다.

---

## 9. 기존 run_id가 존재할 경우

이미 존재하는 run_id 폴더가 있을 경우:

- 기존 파일을 덮어쓰기 전에 반드시 읽고 상태를 확인합니다
- 중복 생성을 피합니다
- 필요한 파일만 생성합니다
- 다른 run_id 폴더의 파일은 절대 수정하지 않습니다

---

## 10. 절대 금지 사항

다음은 절대 금지됩니다:

- outputs 루트에 직접 파일 생성
- JSON 대신 긴 텍스트 보고서 생성 (report-writer 제외)
- 다른 run_id 폴더 수정
- _manifest.json 업데이트 누락
- reference/ 외 위치에 JSON 산출물 생성

---

# 목표

이 워크플로우의 목표는 다음과 같습니다:

- 분석과 보고를 완전히 분리
- 토큰 사용 최소화
- 실행 단위별 완전한 독립성 유지
- 안정적인 보고서 생성