# LLM Prompts (JSON only)

## 공통 규칙
- 출력은 무조건 JSON만 (설명 문장 금지)
- 스키마에 없는 키 생성 금지
- 민감 단어는 soft_replace 룰을 따라 완곡 처리 제안

---

## 1) Idea Enrich (hooks/angles/risks)
Output schema:
{
  "summary": "string",
  "generated_hooks": ["string x10"],
  "generated_angles": ["string x5"],
  "risk_flags": {
    "copyright": "low|medium|high",
    "sensitive": ["string"],
    "notes": "string"
  }
}

---

## 2) Style Profile Extract
Output schema:
{
  "sentence_count": {"min": 0, "max": 0},
  "ending_styles": {"-임": 0.0},
  "connectors": ["string"],
  "hook_types_ranked": ["string"],
  "cta_types_ranked": ["string"],
  "taboo_rules": {
    "hard_block": ["string"],
    "soft_replace": {"원단어":"대체어"}
  }
}

---

## 3) Script Generate
Output schema:
{
  "duration_target": 60,
  "title_candidates": ["string"],
  "script": "string",
  "timeline": [
    {"sec":"0-2","narration":"...","on_screen_text":"...","visual":"..."}
  ],
  "cta": {"type":"question|subscribe|comment|none","line":"string"}
}

---

## 4) JP 3-line Translate
Output schema:
{
  "lines": [
    {"jp":"...","pron":"...","ko":"..."}
  ],
  "jp_tts_only": "string"
}
