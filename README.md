# ResQ-AI-Emergency-Response-Network

A working synthetic emergency intelligence MVP: dark geographic console, authenticated reports, explainable classification, human review, and audit history.

**RESQ is not a replacement for 911, emergency services, or official alerts. All MVP reports are synthetic. No autonomous dispatch is implemented. Never enter real personal or sensitive location information.**

## Quick start

Install Docker with Compose. Copy `.env.example` to `.env`, replace the database password with a random URL-safe value, then run `docker compose up --build`. Open http://localhost:3000 and choose a demo role, then Enter demo. API documentation: http://localhost:8000/docs; machine-readable specification: http://localhost:8000/openapi.json.

Demo mode explicitly enables publicly known demo credentials and seeds six simulated reports. Keep it local. Default application auth is fail closed; Compose defaults demo mode off unless configured in `.env`.

## Architecture

Next.js / React / TypeScript / Tailwind and Leaflet → FastAPI → PostgreSQL. PostGIS extension and spatial-index schema are included. The application stores rounded decimal coordinates; the optional spatial projection table is reserved for future geographic queries. WebSocket notifications invalidate the authenticated incident list every ten seconds. They contain no report contents. Supabase JWTs are verified against JWKS with issuer and audience checks. Trusted `app_metadata.resq_role` controls reviewer/admin access; never derive roles from user-editable metadata.

Submission → bounded text and consent validation → per-user rate limit → moderation hook → deterministic classification → pending queue → authorized human publish/reject → transactional audit. Confidence is a heuristic, not a calibrated probability. Published means reviewed simulation, not an official verified alert.

## Local development

Backend: create a Python 3.12 virtual environment, install `backend/requirements.txt`, then from `backend` run `DEMO_MODE=true uvicorn app.main:app --reload`. SQLite is the local fallback. Frontend: from `frontend`, run `npm install`, then `NEXT_PUBLIC_DEMO_MODE=true npm run dev`. Use `NEXT_PUBLIC_API_URL` to change the service endpoint.

Tests: from `backend`, `python -m pytest -q`. Frontend: `npm run typecheck` and `npm run build`. CI runs API tests and a production web build.

## Supabase / deployment

Set `DEMO_MODE=false`; set API `SUPABASE_URL`, `DATABASE_URL`, and explicit `CORS_ORIGINS`. Set frontend build-time `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `NEXT_PUBLIC_API_URL`, and `NEXT_PUBLIC_DEMO_MODE=false`. Configure Supabase email redirect URLs for your HTTPS origin. Assign trusted roles through server administration only. Frontend environment variables must be present during Next.js build; adapt the Docker build arguments for your deployment. Never expose a Supabase service role key.

Deploy behind HTTPS/WSS with managed Postgres, backups, connection limits, and an ingress rate limiter. Compose binds local ports only. Use a secrets manager. Apply reviewed migrations instead of startup create_all before operational use. Use a shared rate-limit store for multiple workers; the MVP limiter is process-local. Set retention and audit export policies, monitor errors, and complete security/privacy review before any real-world trial.

## MVP boundaries

Text-only synthetic submission; image/voice ingestion and real AI providers are not implemented. No dispatch, resource allocation, official feeds, or sensitive locations. Category filtering works; marker clustering and regional filters are future work. Admins share reviewer capabilities; audit data is available through the protected API. Local drafts persist in browser storage and export as JSON; they are not encrypted and should contain simulation data only. Map tiles require internet access; offline support covers draft editing/export, not map caching. The WebSocket is a minimal public refresh channel, not a durable event bus. Replace keyword rules and moderation hooks with evaluated providers before expanding scope.
