---
name: pdf-generate
description: 보고서 내용을 PDF로 저장합니다. 프로젝트에 포함된 폰트를 사용하여 한글 깨짐을 방지합니다.
---

# PDF 생성 스킬

## 목적

보고서 내용을 PDF로 변환하여 `outputs/{run_id}/` 경로에 저장합니다.

## 폰트 경로

이 스킬과 같은 폴더에 폰트가 포함되어 있습니다. 네트워크나 시스템 폰트에 의존하지 않습니다.

```
.claude/skills/pdf-generate/
├── SKILL.md
└── fonts/
    └── NotoSansKR-VariableFont_wght.ttf
```

폰트 경로는 항상 이 파일(`SKILL.md`)의 위치를 기준으로 계산합니다:

```python
import os
SKILL_DIR = os.path.dirname(os.path.abspath(__file__))
FONT_PATH = os.path.join(SKILL_DIR, "fonts", "NotoSansKR-VariableFont_wght.ttf")
```

단, Claude가 직접 실행하는 경우 `__file__`을 쓸 수 없으므로 아래처럼 프로젝트 루트 기준으로 계산합니다:

```python
import os
PROJECT_ROOT = "/path/to/accounting-review-team-main"  # 실행 환경의 실제 경로로 대체
FONT_PATH = os.path.join(PROJECT_ROOT, ".claude", "skills", "pdf-generate", "fonts", "NotoSansKR-VariableFont_wght.ttf")
```

## 실행 절차

### 1. 의존성 확인

```bash
pip install reportlab --break-system-packages
```

reportlab은 대부분의 환경에 사전 설치되어 있으므로 설치가 필요 없는 경우가 많습니다.

### 2. 폰트 등록 및 PDF 생성

```python
from reportlab.pdfbase import pdfmetrics
from reportlab.pdfbase.ttfonts import TTFont
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, Table, TableStyle
from reportlab.lib.pagesizes import A4
from reportlab.lib.styles import ParagraphStyle
from reportlab.lib import colors
import os

# 폰트 등록
FONT_PATH = os.path.join(PROJECT_ROOT, ".claude", "skills", "pdf-generate", "fonts", "NotoSansKR-VariableFont_wght.ttf")
pdfmetrics.registerFont(TTFont("NotoSansKR", FONT_PATH))
FONT = "NotoSansKR"

# 문서 생성
output_path = f"outputs/{run_id}/{run_id}-경영자보고서.pdf"
doc = SimpleDocTemplate(output_path, pagesize=A4,
                        rightMargin=18*mm, leftMargin=18*mm,
                        topMargin=20*mm, bottomMargin=20*mm)

# 스타일 정의 예시
body_style = ParagraphStyle("body", fontName=FONT, fontSize=10, leading=16)

# 내용 구성 후 빌드
story = [...]  # Paragraph, Table, Spacer 등
doc.build(story)
```

## 출력

- `outputs/{run_id}/{run_id}-경영자보고서.pdf`

## 주의사항

- 숫자/금액은 `1,234,567원` 형태로 표준화합니다.
- 특수 문자(희귀 이모지 등)는 사용하지 않습니다.
- 표의 열 수가 많으면 부록으로 이동합니다.
- `TTFont`로 등록한 폰트는 모든 한글 글리프를 포함하므로 깨짐이 발생하지 않습니다.
