# API Contract (MVP v0.1)

Base URL: /api/v1

## Common
### GET /health
Response 200:
{ "ok": true }

---

## Ideas
### POST /ideas
Request:
{
  "project_id": "uuid",
  "source_type": "text|link|upload",
  "title": "optional",
  "raw_text": "optional",
  "url": "optional",
  "tags": ["optional", "array"]
}

Response 201:
{
  "id": "uuid",
  "project_id": "uuid",
  "source_type": "text",
  "title": "...",
  "raw_text": "...",
  "url": null,
  "tags": [],
  "risk_flags": {},
  "generated_hooks": [],
  "generated_angles": [],
  "summary": null,
  "created_at": "...",
  "updated_at": "..."
}

### POST /ideas/{idea_id}/enrich
Goal: 훅 10개 + 각도 5개 + 요약 + 리스크 태그 생성

Request:
{
  "style_hint": "optional (예: 분노형/팩트형/유머형)",
  "language": "ko"
}

Response 200:
{
  "id": "uuid",
  "summary": "1-2문장 요약",
  "generated_hooks": ["...x10"],
  "generated_angles": ["...x5"],
  "risk_flags": {
    "copyright": "low|medium|high",
    "sensitive": ["optional list"],
    "notes": "optional"
  }
}

---

## Style Profiles
### POST /style-profiles/extract
Goal: 벤치마킹 대본들로 rules_json 생성

Request:
{
  "workspace_id": "uuid",
  "name": "예: 커뮤니티_분노형",
  "benchmark_scripts": [
    "script text 1",
    "script text 2"
  ],
  "language": "ko"
}

Response 201:
{
  "id": "uuid",
  "workspace_id": "uuid",
  "name": "...",
  "sample_count": 10,
  "rules_json": {
    "sentence_count": {"min": 12, "max": 16},
    "ending_styles": {"-임": 0.3, "-지": 0.2, "-는데": 0.2, "-다": 0.3},
    "connectors": ["하지만","심지어","참고로","게다가"],
    "hook_types_ranked": ["궁금증","충격","비교","질문"],
    "cta_types_ranked": ["질문형","댓글유도","구독유도"],
    "taboo_rules": {
      "hard_block": [],
      "soft_replace": {"살인":"사건","마약":"불법 약물"}
    }
  },
  "created_at": "...",
  "updated_at": "..."
}

### GET /style-profiles?workspace_id=uuid
Response 200:
{ "items": [ ...style profiles... ] }

---

## Scripts
### POST /scripts/generate
Request:
{
  "project_id": "uuid",
  "idea_id": "uuid",
  "style_profile_id": "uuid",
  "duration_target": 60,
  "language": "ko",
  "cta_mode": "question|subscribe|comment|none"
}

Response 201:
{
  "id": "uuid",
  "project_id": "uuid",
  "idea_id": "uuid",
  "style_profile_id": "uuid",
  "duration_target": 60,
  "language": "ko",
  "title_candidates": ["..."],
  "script_text": "...",
  "timeline_json": [
    {"sec":"0-2","narration":"...","on_screen_text":"...","visual":"..."}
  ],
  "analysis_json": {
    "sentence_count": 14,
    "avg_sentence_len": 42,
    "ending_styles": {"-임": 5, "-지": 3},
    "connectors": {"하지만": 2, "심지어": 1},
    "has_cta": true,
    "hook_strength_notes": ["..."],
    "risk_flags": {"sensitive_terms": ["..."]}
  },
  "version": 1,
  "status": "draft",
  "created_at": "...",
  "updated_at": "..."
}

### POST /scripts/{script_id}/analyze
Request:
{ "style_profile_id": "uuid" }

Response 200:
{ "analysis_json": { ...same schema... } }

### GET /scripts?project_id=uuid
Response 200:
{ "items": [ ...scripts... ] }

---

## Exports
### POST /exports/tts
Request:
{ "script_id": "uuid" }

Response 200:
{
  "export_type": "tts",
  "payload_text": "부호 제거 + 문장 분할된 TTS용 텍스트"
}

### POST /exports/srt
Request:
{
  "script_id": "uuid",
  "max_chars_per_line": 7
}

Response 200:
{
  "export_type": "srt",
  "payload_text": "SRT text",
  "payload_json": {
    "segments": [
      {"start":"00:00:00,000","end":"00:00:01,500","text":"..."}
    ]
  }
}

### POST /exports/translate/jp-3line
Request:
{ "script_id": "uuid" }

Response 200:
{
  "export_type": "jp_3line",
  "payload_text": "일본어/발음/뜻 3줄 포맷 전체",
  "payload_json": {
    "lines": [
      {"jp":"...","pron":"...","ko":"..."}
    ],
    "jp_tts_only": "일본어만 모아둔 버전"
  }
}

---

## SOP
### POST /sops/generate
Request:
{
  "project_id": "uuid",
  "workspace_id": "uuid",
  "template_hint": "예: 커뮤니티썰 템플릿",
  "reference_links": ["optional list"],
  "rules": {
    "subtitle_max_chars": 7,
    "avoid_words": ["..."],
    "cta_required": true
  }
}

Response 201:
{
  "id": "uuid",
  "doc_markdown": "# 작업지시서...\n..."
}
