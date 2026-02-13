# ShortOps (MVP v0.1) 제품 명세

## 1) 제품 목표 (MVP)
유튜브 쇼츠 제작 운영을 **벤치마킹 분해 → 공식화 → 소재 → 대본 → 내보내기(TTS/SRT/번역) → SOP(작업지시서)** 흐름으로 자동화한다.

### 핵심 사용자 가치
- 벤치마킹 대본을 넣으면 말투/구조 규칙(Style Profile)을 추출한다.
- 소재(텍스트/링크)를 넣으면 대본을 생성한다. (15/30/60/180초)
- 대본을 TTS/SRT/일본어 3줄 번역 포맷으로 내보낸다.
- 프리랜서에게 전달할 SOP(작업지시서)를 원클릭 생성한다.

## 2) MVP 범위 (In Scope)
- Idea Inbox (소재 저장 + 훅/각도/리스크 태그 생성)
- Style Profile 추출 (벤치마킹 대본 10개 권장)
- Script Studio (대본 생성 + 간단 분석 룰 엔진)
- Export (TTS용 정리본 / SRT 초안 / 일본어 3줄 번역)
- SOP Generator (프리랜서 작업지시서 문서 생성)

## 3) MVP 제외 (Out of Scope)
- 계정/인증/우회/다운로드 등 플랫폼 제한 우회 기능
- “경쟁강도 점수(0~100)” 같은 과장 점수 제공
  - 대신 데이터 스냅샷은 v1.1에서 고려

## 4) 권장 기술 스택
### Backend
- FastAPI + Pydantic
- SQLAlchemy 2.x + Alembic migrations
- PostgreSQL
- (옵션) Redis (캐시/레이트리밋)

### Frontend
- Next.js (App Router)
- 최소 UI: Inbox / Script Studio / Export / SOP

### LLM
- OpenAI Responses API 사용
- 모델은 환경변수로 지정 (기본값: `gpt-5.2-codex` 권장)
- 실제 모델명은 환경변수로 교체 가능하게 설계

## 5) 레포 구조 (목표 트리)
```text
shortops/
  api/
    app/
      main.py
      core/
        config.py
        db.py
        errors.py
      llm/
        client.py
        prompts.py
        schemas.py
      models/
        base.py
        user.py
        workspace.py
        project.py
        idea_item.py
        style_profile.py
        script.py
        export.py
        sop.py
      routers/
        health.py
        ideas.py
        style_profiles.py
        scripts.py
        exports.py
        sops.py
      services/
        idea_service.py
        style_profile_service.py
        script_service.py
        export_service.py
        sop_service.py
      rules/
        script_analysis.py
    alembic/
    tests/
    pyproject.toml
  web/
    app/
      layout.tsx
      page.tsx
      inbox/page.tsx
      studio/page.tsx
      export/page.tsx
      sop/page.tsx
    components/
    lib/api.ts
    package.json
  docs/
    SPEC.md
    DB_SCHEMA.sql
    API_CONTRACT.md
    LLM_PROMPTS.md
  docker-compose.yml
  .env.example
  README.md
```

## 6) 데이터 모델 개요
Workspace 중심 구조로 설계하며, MVP에서는 인증을 최소화할 수 있다.

핵심 엔티티:
- `workspaces`
- `projects` (workspace 소속)
- `idea_items` (project 소속)
- `style_profiles` (workspace 소속)
- `scripts` (project 소속, `idea_item` + `style_profile` 참조)
- `exports` (script 소속)
- `sop_templates` (workspace 소속)
- `sops` (project 소속)

## 7) 공통 에러 응답 포맷
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Human readable message",
    "details": {}
  }
}
```

## 8) API 요약
- `GET /api/v1/health`

### Ideas
- `POST /api/v1/ideas`
- `POST /api/v1/ideas/{idea_id}/enrich`

### Style Profiles
- `POST /api/v1/style-profiles/extract`
- `GET /api/v1/style-profiles`

### Scripts
- `POST /api/v1/scripts/generate`
- `POST /api/v1/scripts/{script_id}/analyze`
- `GET /api/v1/scripts?project_id=...`

### Exports
- `POST /api/v1/exports/tts`
- `POST /api/v1/exports/srt`
- `POST /api/v1/exports/translate/jp-3line`

### SOP
- `POST /api/v1/sops/generate`

## 9) 룰 엔진 (대본 분석)
MVP 분석은 코드 기반 규칙으로 구현하고 LLM 의존을 최소화한다.

필수 규칙:
- 문장 수
- 평균 문장 길이(글자 수)
- 어미(종결 패턴) 분포 대략 추정
- 접속사/전환어 카운트
- 훅(첫 1~2문장) 트리거 탐지
- CTA(질문형/구독/댓글 유도) 탐지
- 민감 단어 탐지(커스텀 사전)

## 10) UI (최소)
- `/inbox`: 소재 추가, 리스트 조회, enrich 실행
- `/studio`: idea 선택 → style profile 선택 → duration 선택 → generate
- `/export`: script 선택 → tts/srt/jp export
- `/sop`: project 선택 → SOP 생성

## 11) 개발 원칙
- LLM 출력은 **JSON only** + 스키마 검증 (실패 시 재시도 1회)
- DB 마이그레이션 필수
- 최소 단위 테스트 (룰 엔진/서비스 레이어)
