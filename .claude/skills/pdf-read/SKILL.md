---
name: pdf-read
description: 사용자가 업로드한 파일이 PDF인 경우 텍스트를 추출합니다. PDF 파일이 입력으로 주어졌을 때만 사용합니다.
---

# PDF 읽기 스킬

## 사용 조건

사용자가 업로드한 파일의 확장자가 `.pdf`인 경우에만 이 스킬을 사용합니다.

## 목적

PDF 파일에서 텍스트를 추출하여 에이전트가 내용을 분석할 수 있도록 합니다.

## 실행 절차

### 1. 의존성 확인 및 설치

```bash
pip install pdfplumber --break-system-packages
```

### 2. 텍스트 추출

```python
import pdfplumber

with pdfplumber.open("업로드된파일.pdf") as pdf:
    text = ""
    for page in pdf.pages:
        extracted = page.extract_text()
        if extracted:
            text += extracted + "\n"

print(text)
```

### 3. 표 추출 (표가 포함된 경우)

```python
with pdfplumber.open("업로드된파일.pdf") as pdf:
    for i, page in enumerate(pdf.pages):
        tables = page.extract_tables()
        for table in tables:
            print(f"[페이지 {i+1} 표]")
            for row in table:
                print(row)
```

## 출력

- 추출된 텍스트를 에이전트 컨텍스트로 전달합니다.
- 별도 파일로 저장하지 않습니다.

## 주의사항

- 스캔된 이미지 기반 PDF는 텍스트 추출이 되지 않을 수 있습니다.
- 한글이 포함된 PDF도 pdfplumber로 정상 추출됩니다.
