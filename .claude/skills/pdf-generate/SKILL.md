---
name: pdf-generate
description: 보고서 내용을 HTML로 변환한 뒤 PDF로 저장합니다. 한글 폰트를 적용하여 깨짐을 방지합니다.
---

# PDF 생성 스킬

## 목적

보고서 내용을 HTML로 렌더링한 후 PDF로 변환하여 `outputs/{run_id}/` 경로에 저장합니다.  
HTML은 중간 변환 단계로만 사용하며 최종 저장하지 않습니다.

## 실행 절차

### 1. 의존성 확인 및 설치

```bash
pip install weasyprint --break-system-packages
```

한글 폰트가 없는 경우 설치:

```bash
apt-get install -y fonts-noto-cjk
```

### 2. HTML 생성

보고서 내용을 아래 템플릿 기반으로 HTML 문자열로 작성합니다.

```python
html_content = """
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR&display=swap');

    body {
      font-family: 'Noto Sans KR', 'Noto Sans CJK KR', sans-serif;
      font-size: 12pt;
      line-height: 1.8;
      margin: 40px;
      color: #222;
    }
    h1 { font-size: 20pt; margin-bottom: 8px; }
    h2 { font-size: 15pt; margin-top: 24px; }
    h3 { font-size: 13pt; }
    table {
      border-collapse: collapse;
      width: 100%;
      margin: 16px 0;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 6px 10px;
      text-align: left;
    }
    th { background-color: #f0f0f0; }
  </style>
</head>
<body>
  <!-- 보고서 내용 삽입 -->
  {content}
</body>
</html>
"""
```

### 3. PDF 변환 및 저장

```python
from weasyprint import HTML
import os

run_id = "실행ID"  # run_id를 변수로 받아서 사용
output_dir = f"outputs/{run_id}"
os.makedirs(output_dir, exist_ok=True)

output_path = f"{output_dir}/{run_id}-경영자보고서.pdf"

HTML(string=html_content).write_pdf(output_path)
print(f"PDF 저장 완료: {output_path}")
```

## 출력

- `outputs/{run_id}/{run_id}-경영자보고서.pdf`

## 주의사항

- 숫자/금액은 `1,234,567원` 형태로 표준화합니다.
- 특수 문자(희귀 이모지 등)는 사용하지 않습니다.
- 표의 열 수가 많으면 부록으로 이동합니다.
- 오프라인 환경에서는 Google Fonts 대신 시스템에 설치된 `Noto Sans CJK KR` 폰트가 자동 적용됩니다.
