# ShortOps MVP PRD (운영자/양산자 타깃)

## 1) 제품 개요

### 1.1 제품 한 줄 정의
**ShortOps**는 벤치마킹을 규칙으로 저장하고, 해당 규칙 기반으로 `소재 → 대본 → 내보내기 → SOP`를 연결하는 쇼츠 운영 SaaS다.

### 1.2 타깃 사용자
- 1차 타깃: 프리랜서를 활용해 쇼츠를 양산하는 운영자
- 핵심 니즈:
  - 생산 속도 향상
  - 품질 표준화
  - 정책/리스크 사전 점검

### 1.3 핵심 엔진(3 Engine)
1. **Bench Engine**: 대본/구성 분해 + 규칙화 + 재사용
2. **Idea Engine**: 소재 인입 → 정리 → 우선순위화 → 대본 연동
3. **Script Engine**: 훅-전개-결론-CTA 자동 생성 및 룰 검사

---

## 2) MVP 범위

### 2.1 포함 기능 (Must)
1. **Script Studio**
   - 대본 생성 (15/30/60/180초)
   - 대본 분석(문장 길이, 어미, 접속사, CTA, 민감어)
2. **Benchmark Analyzer / Style Profile**
   - 벤치마킹 대본 업로드
   - 규칙 자동 추출 및 템플릿 저장
3. **Idea Inbox**
   - 링크/텍스트/메모 소재 수집
   - 훅/각도/리스크 태깅 자동 생성
4. **Export**
   - TTS 정리본
   - SRT 초안(문장 단위)
   - 일본어 3줄 포맷(원문/발음/뜻)
5. **SOP Generator**
   - 프리랜서 작업지시서 자동 생성(PDF/Doc/복사용)

### 2.2 제외 기능 (Out of Scope)
- 계정 우회/인증 우회/스팸 회피 관련 기능
- 정책 위반 유도 자동화
- 영상 무단 다운로드/무단 크롤링
- 커스텀 경쟁 점수(0~100) 중심 랭킹

---

## 3) 정보구조(IA)

- Dashboard
- Idea Inbox
- Script Studio
- Templates
  - Style Profiles
  - Script Templates
  - SOP Templates
- Export
- Settings (팀/권한/청구/모델 키)

### 3.1 핵심 사용자 플로우
1. 벤치마킹 대본 업로드 → Style Profile 생성
2. Idea Inbox에서 소재 선택 → Script Studio에서 대본 4종 생성
3. Export에서 TTS/SRT/번역 산출
4. SOP 생성 → 프리랜서 전달

---

## 4) 기능 요구사항

## 4.1 Script Studio
**입력**
- 소재 텍스트/링크
- Style Profile
- 길이 타깃(15/30/60/180)
- 수위 정책(민감어 제한 수준)
- CTA 타입

**출력**
- 대본 본문
- 타임라인 가이드(컷/자막)
- 체크리스트 리포트

## 4.2 Benchmark Analyzer
**입력**
- 벤치마킹 대본 10개 이상 권장

**출력**
- 문장 수 범위
- 문장 길이 특성
- 어미 분포
- 접속사 사전
- 훅/CTA 유형 분포
- 금칙어/완곡어 규칙

## 4.3 Idea Inbox
**입력**
- 텍스트/링크/메모
- 태그(카테고리/사건/플랫폼)

**자동 생성**
- 1문장 요약
- 훅 5~10개
- 확장 각도 5개
- 리스크 라벨(저작권/폭력/혐오/민감)

## 4.4 Export
- TTS용 대본: 부호 제거 + 문장 분할
- SRT 초안: 문장 단위 자막
- 일본어 3줄 포맷: 일본어/발음/뜻

## 4.5 SOP Generator
**입력**
- 레퍼런스 링크
- 자막 규칙(글자수, 폰트, 색)
- 컷 규칙(컷 길이, 전환)
- 금칙어/편집 주의
- 납기/검수 기준

**출력**
- 작업지시서(문서)
- 테스트 과제 초안

---

## 5) 정책/컴플라이언스 요구사항

1. 정책 준수 가이드를 제공하되, **우회 기능은 미제공**
2. 경쟁 분석은 **점수화보다 지표 리포트 중심**
3. YouTube API 연동 시:
   - OAuth 2.0 사용
   - 사용자 철회 시 저장 데이터 삭제 프로세스 보장
   - 호출 쿼터 모니터링 및 캐시 설계
4. 검색 호출 최소화(search.list 고비용 고려)

---

## 6) 기술 설계

## 6.1 아키텍처
- Frontend: Next.js
- Backend: FastAPI 또는 NestJS
- DB: PostgreSQL(+ pgvector)
- Queue/Cache: Redis
- Storage: S3 호환 스토리지
- LLM 계층:
  - 생성: 대본/훅/번역
  - RAG: 전자책/내부 가이드 근거 회수
  - 규칙 엔진: 코드 기반 체크(비LLM)

## 6.2 데이터 모델 (최소)
- users
- workspaces, workspace_members
- projects
- idea_items
- style_profiles
- scripts
- exports
- sop_templates, sops

---

## 7) API 초안

### Script
- `POST /scripts/generate`
- `POST /scripts/analyze`
- `POST /scripts/rewrite`

### Style
- `POST /style-profiles/extract`

### Idea
- `POST /ideas`
- `POST /ideas/enrich`

### Export
- `POST /export/tts`
- `POST /export/srt`
- `POST /export/translate/jp-3line`

### SOP
- `POST /sop/generate`

---

## 8) 체크리스트 룰 엔진 정의

### 8.1 분석 항목
- 문장 수
- 평균 문장 길이/편차
- 어미 분포(예: -임/-지/-는데요/-다)
- 접속사 빈도
- 훅 신호어(첫 1~2문장)
- CTA 존재/유형
- 민감어 탐지 및 완곡어 제안

### 8.2 출력 원칙
- 단일 점수 대신 **차이 리포트** 중심
- Style Profile 대비 과다/과소 사용 항목 표시

---

## 9) 단계별 개발 우선순위

1. Script Studio + 룰 엔진
2. Style Profile 추출/적용
3. Idea Inbox
4. Export
5. SOP Generator
6. Competition Snapshot (지표 리포트형, 캐시 필수)

---

## 10) 성공 지표 (MVP)
- 대본 생성 후 수정 시간 30% 이상 감소
- 팀 내 템플릿 재사용률 60% 이상
- SOP 생성 후 외주 커뮤니케이션 왕복 횟수 감소
- Export 기능 사용률(프로젝트당 1회 이상)

