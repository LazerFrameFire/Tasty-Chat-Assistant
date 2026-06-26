# Tasty

A dark-mode food taste assistant powered by OpenAI. Chat with "Taster" to discover what any food tastes like, save your favorites, and get personalized predictions.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/tasty run dev` — run the frontend (port 26041)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string (auto-provisioned)
- Required env: `OPENAI_API_KEY` — OpenAI API key (set in Secrets)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite, Tailwind CSS, shadcn/ui, wouter, TanStack Query
- API: Express 5
- DB: PostgreSQL + Drizzle ORM (for liked foods)
- AI: OpenAI Responses API (`client.responses.create()`) with file_search + vector store
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/tasty/src/pages/Home.tsx` — main chat page
- `artifacts/tasty/src/index.css` — dark theme (amber/terracotta palette)
- `artifacts/api-server/src/routes/chat.ts` — streaming + non-streaming OpenAI chat routes
- `artifacts/api-server/src/routes/foods.ts` — liked foods CRUD + daily food + predictions
- `artifacts/api-server/config.json` — `assistant_name`, `vector_store_id`, `model` (from project zip)
- `lib/db/src/schema/likedFoods.ts` — liked_foods table
- `lib/api-spec/openapi.yaml` — API contract source of truth

## Architecture decisions

- Uses OpenAI **Responses API** (`client.responses.create()`), NOT the deprecated Assistants API
- Streaming via SSE on `/api/chat/stream` — frontend uses `fetch` + `ReadableStream`, not generated hooks
- Vector store loaded from `config.json` (pre-existing corpus, never rebuilt)
- `previous_response_id` chained per conversation turn for context continuity
- Food predictions use a separate GPT-4o-mini call prompted to return a JSON array

## Product

- Dark mode chat UI with Taster welcome screen and 4 suggested prompts
- Streaming responses with animated loading indicator
- Expandable citation sources on assistant messages
- Daily food pick (rotates by day-of-year)
- Saved Tastes sidebar with add/remove, persisted in PostgreSQL
- Personalized "You might also like" predictions based on saved foods

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Always run codegen after editing `lib/api-spec/openapi.yaml`
- `config.json` must stay at `artifacts/api-server/config.json` — path resolved relative to workspace root
- The streaming endpoint (`/api/chat/stream`) is not in the OpenAPI spec — frontend calls it directly with `fetch`
- Google Fonts `@import url(...)` must be the very first line of `index.css`

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
