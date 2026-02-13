# ShortOps MVP - Issue List (Codex tasks)

## 사용법
- Codex에게 "이 이슈 하나만" 처리하게 시킨다.
- 처리 후: 테스트/실행 커맨드까지 돌리고 결과 요약하게 시킨다.

---

## Issue 01 - Repo scaffold (api + web)
Goal:
- FastAPI / Next.js 기본 앱이 실행되는 상태

Acceptance:
- docker-compose up 하면 postgres 뜸
- api: /api/v1/health OK
- web: /inbox 페이지 렌더링

Codex Prompt:
"Create the repository scaffold exactly as docs/SPEC.md. 
Implement api (FastAPI) and web (Next.js) folders. 
Add minimal health endpoint /api/v1/health and web pages /inbox /studio /export /sop.
Add README with local run steps.
Use Python uv + pyproject, and Node with package.json.
Run basic sanity checks."

---

## Issue 02 - Database + Alembic migrations
Goal:
- Postgres 연결 + 마이그레이션 세팅 + 테이블 생성

Acceptance:
- `alembic upgrade head` 성공
- docs/DB_SCHEMA.sql과 동일한 테이블 구조가 생성됨

Codex Prompt:
"Implement SQLAlchemy models and Alembic migrations per docs/DB_SCHEMA.sql.
Add app/core/db.py for engine/session.
Add `alembic revision --autogenerate` workflow and docs in README.
Include a simple seed script to create a default user/workspace/project for dev."

---

## Issue 03 - Pydantic schemas + error format 통일
Goal:
- API 요청/응답 스키마 고정
- 에러 포맷 통일

Acceptance:
- Validation 에러가 {error:{code,message,details}} 형태로 응답
- docs/API_CONTRACT.md에 맞는 스키마가 구현됨

Codex Prompt:
"Create Pydantic request/response schemas matching docs/API_CONTRACT.md.
Implement consistent error handling middleware that returns the unified error format.
Add minimal unit tests for error formatting."

---

## Issue 04 - Idea endpoints (create + enrich)
Goal:
- POST /ideas
- POST /ideas/{id}/enrich (LLM은 mock부터)

Acceptance:
- idea 생성/조회 가능
- enrich 호출 시 summary/hooks/angles/risk_flags가 DB에 저장됨

Codex Prompt:
"Implement idea_items CRUD (create + list by project).
Implement enrich endpoint: first use a deterministic mock generator (no OpenAI yet).
Store generated fields in DB.
Write tests for enrich persistence."

---

## Issue 05 - Style Profile extract endpoint (mock)
Goal:
- POST /style-profiles/extract (mock)

Acceptance:
- benchmark_scripts 입력하면 rules_json 생성 + 저장

Codex Prompt:
"Implement style profile extract endpoint.
For MVP phase 1, mock rules_json generation deterministically (counts + simple heuristics).
Store sample_count and rules_json.
Add GET /style-profiles?workspace_id=... list endpoint."

---

## Issue 06 - Script rules engine (no LLM)
Goal:
- 대본 분석 룰 엔진 구현 (문장수/길이/어미/접속사/CTA/민감어)

Acceptance:
- script_analysis.py에 pure function으로 구현
- 입력 대본에 대해 analysis_json 생성

Codex Prompt:
"Implement app/rules/script_analysis.py.
Add functions:
- split_sentences_ko
- calc_sentence_count, avg_len
- detect_endings (approx by suffix patterns)
- count_connectors
- detect_hook
- detect_cta
- detect_sensitive_terms (configurable list)
Add unit tests."

---

## Issue 07 - Script generate endpoint (mock -> rules engine 연동)
Goal:
- /scripts/generate로 script 생성하고 analysis_json 채움

Acceptance:
- 저장된 script가 /scripts?project_id=...에서 조회
- analysis_json이 항상 포함

Codex Prompt:
"Implement script generation service with mock output (based on idea summary + style rules).
After generating script_text, run rules engine to produce analysis_json.
Store title_candidates and timeline_json as simple placeholders."

---

## Issue 08 - Export endpoints (tts/srt/jp-3line) - mock
Goal:
- TTS용 텍스트 변환
- SRT 생성(간단 분절)
- JP 3줄 번역(모킹)

Acceptance:
- 각 export 결과가 exports 테이블에 저장

Codex Prompt:
"Implement exports:
- tts: remove punctuation, split into short lines
- srt: generate dummy timings and wrap text by max_chars_per_line
- jp_3line: mock translation (placeholder) but keep schema
Store outputs in exports table."

---

## Issue 09 - OpenAI Responses API 붙이기 (mock 대체)
Goal:
- Issue 04/05/07/08의 mock을 OpenAI 호출로 교체

Acceptance:
- OPENAI_API_KEY 없으면 mock fallback
- 있으면 Responses API 호출로 JSON output 생성
- JSON schema validation 실패 시 1회 재시도

Codex Prompt:
"Integrate OpenAI Responses API in app/llm/client.py and replace mock generators.
Use env OPENAI_MODEL (default gpt-5.2-codex).
Enforce JSON-only outputs + Pydantic validation.
Retry once on schema validation failure.
Keep fallback to mock if key missing."

---

## Issue 10 - Web UI wiring (real API)
Goal:
- web에서 실제 API 호출로 전체 플로우 동작

Acceptance:
- /inbox에서 idea 생성 + enrich
- /studio에서 script 생성
- /export에서 tts/srt/jp 내보내기
- /sop에서 SOP 생성

Codex Prompt:
"Wire Next.js pages to call API endpoints.
Provide basic forms and result panels with copy-to-clipboard.
Add minimal loading/error states.
No auth for MVP; use a fixed workspace/project seed."

---

## Issue 11 (Optional) - SOP Generator
Goal:
- /sops/generate에서 마크다운 작업지시서 생성

Acceptance:
- 생성된 doc_markdown이 DB에 저장
- web에서 복사 가능

Codex Prompt:
"Implement SOP generator endpoint.
Given template_hint + rules + reference_links, output a clean markdown SOP document.
Store it in sops table. Display in /sop page."
