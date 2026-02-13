# ShortOps MVP

## Prereqs
- Node 18+
- Python 3.11+
- Docker

## Run Postgres
docker compose up -d

## Backend
cd api
# (Codex가 uv/poetry 중 하나로 세팅해줄 것)
# 예: uv sync / pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000

Health:
GET http://localhost:8000/api/v1/health

## Frontend
cd web
npm install
npm run dev

Open:
http://localhost:3000/inbox
