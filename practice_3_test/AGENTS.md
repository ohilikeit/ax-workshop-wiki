# AGENTS.md — 캐치 공고 변환 웹앱 가이드

이 프로젝트에서 **일관된 결과**를 만들도록 한 가이드라인입니다. 작업 시작 전 반드시 이 파일을 먼저 읽고, 아래 규칙을 모든 결과물에 적용하세요. 학습자(비개발자)는 이 파일을 건드리지 않습니다.

---

## 0. 프로젝트 목표

외부 채용공고 이미지 → 캐치(catch.co.kr) 공고 등록 폼 양식의 엑셀 + HTML로 변환하는 웹앱. 좌측 사이드바에서 이미지 업로드, 메인 영역은 좌(원본) ↔ 우(결과 탭: 엑셀·HTML·원본 데이터).

---

## 1. 기술 스택 (변경 금지)

| 계층 | 도구 |
|---|---|
| LLM 모델 | `gemini-3-flash-preview` (상수 `MODEL_ID` 로 한 군데만 관리) |
| LLM SDK | `google-genai` — 구 SDK `google-generativeai` 절대 금지 |
| 백엔드 | FastAPI + Python 3.11+ (`uv` 권장, `pip` 도 가능) |
| 프론트 | Next.js 15 App Router + TypeScript + Tailwind |
| 이미지 | Pillow + numpy — OpenCV(cv2) 절대 금지 |
| 출력 | openpyxl (엑셀), Jinja2 (HTML) |

다른 라이브러리 추가는 학습자에게 명시적으로 확인 후 진행. 폴더 구조·파일 분리는 학습자와 코딩 에이전트가 자유롭게 결정합니다.

---

## 2. SDK 사용 패턴 (가장 자주 틀림)

### 올바름 — `google-genai`

```python
from google import genai

client = genai.Client(api_key=API_KEY)

response = client.models.generate_content(
    model=MODEL_ID,   # "gemini-3-flash-preview"
    contents=[image_part, prompt_text],
    config={
        "response_mime_type": "application/json",
        "response_schema": JobPostingSchema,   # Pydantic
    },
)
```

### 금지 — 구 SDK `google-generativeai`

```python
# 이런 패턴이 보이면 즉시 위 새 SDK 패턴으로 다시 작성
import google.generativeai as genai
genai.configure(api_key=...)
model = genai.GenerativeModel("...")
```

import 라인이 `from google import genai` 인지 매번 확인.

---

## 3. 환경 변수

`.env.example` 을 `.env` 로 복사한 뒤 `GEMINI_API_KEY` 값을 채웁니다. Python에선 `python-dotenv` 로 자동 로드 (`from dotenv import load_dotenv; load_dotenv()`). **`.env` 는 절대 git에 커밋하지 않습니다.**

---

## 4. 백엔드 가드레일 (FastAPI)

- **CORS 미들웨어 필수** — 프론트(다른 포트)에서 호출됨
  ```python
  from fastapi.middleware.cors import CORSMiddleware
  app.add_middleware(
      CORSMiddleware,
      allow_origins=["http://localhost:3000"],
      allow_methods=["*"], allow_headers=["*"],
  )
  ```
- **파일 업로드 한도 10MB** — 한화 공고 같은 큰 이미지 대비
- **Structured Output 강제** — AI 호출 시 항상 `response_mime_type="application/json"` + `response_schema=<Pydantic 모델>` 전달
- **카탈로그 주입** — `catalog.md` 내용을 시스템 인스트럭션이나 contents 앞부분에 통째로 주입

---

## 5. 프론트 가드레일 (Next.js 15)

- **App Router 사용** (Pages Router 금지)
- **Server Component 기본**, 인터랙티브(파일 업로드·state·이벤트)만 `'use client'`
- 백엔드 주소는 `NEXT_PUBLIC_API_BASE` 환경변수 (기본 `http://localhost:8000`)
- **Tailwind** 사용 (다른 스타일 라이브러리 금지)
- **파일 업로드 한도 10MB** — `next.config.js` 에서 명시
- **HTML 미리보기는 iframe srcDoc** — 백엔드가 inline CSS 포함된 자체 완결 HTML 반환

---

## 6. 결과 파일 위치 안내 (필수)

모든 실행·CLI 테스트 끝에 **콘솔 마지막에 결과 파일들의 절대 경로**를 출력하세요. 학습자가 탐색기에서 더블클릭으로 확인합니다.

```
변환 완료
  - Excel: /절대/경로/result.xlsx
  - HTML : /절대/경로/result.html

탐색기에서 위 파일을 더블클릭으로 열어 확인해보세요.
```

결과물은 `output/<공고이름>/` 아래에 두는 것을 권장합니다 (공고이름 = 업로드 이미지 파일명에서 확장자 뺀 값). 세부 하위 구조는 자유.

---

## 7. 멀티 포지션 분리 규칙

한 공고에 여러 직군·포지션이 섞여 있을 때:

- **사업부문(division) 단위로 모집부문을 분리** (예: 본사영업·본사지원·디지털금융)
- 사업부문 안의 직무(예: IB·Trading·IT)는 같은 모집부문의 하위 항목으로
- 우대사항이 사업부문별로 다르면 각 모집부문에 분리해 채움
- 카탈로그에 없는 항목은 버리지 말고 `extra_info` 같은 별도 칸으로

---

## 8. 코딩 컨벤션

- **한글 주석 권장** (학습자가 읽을 수 있게)
- 함수·변수명은 영어
- 한 함수에 여러 책임 몰지 말 것 (자르기 / AI 호출 / 양식 채우기 분리)
- 외부 호출(Gemini · 파일 IO · HTTP)은 try/except로 감싸 친절한 에러 메시지

---

## 9. 매 단계 끝 보고 형식 (필수)

단계가 끝나면 반드시 아래 5줄로 보고하세요. 길게 설명하지 마세요.

```
1. 한 일: (이번 단계에서 실제로 한 일 1~2문장)
2. 실행 명령: (실행한 파이썬 명령)
3. 생성/수정 파일: (새로 만들어졌거나 바뀐 파일 목록)
4. 확인할 결과물: (사용자가 직접 열어봐야 할 파일 경로)
5. 다음 단계: (넘어가도 되는지, 아니면 뭘 먼저 확인해야 하는지)
```
