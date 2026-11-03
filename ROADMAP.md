# Roadmap

Spaced-repetition flashcards built with Next.js, Fastify and PostgreSQL.
I use it for German vocabulary and for every interview question I failed.

## How to read this file

- Each phase has **Build**, **Done when**, and an **Interview story** to rehearse aloud in English.
- Tick a box only when the item is done *and verified*: a test passes, a query is measured, a behavior is observed.
- Tags point to the learning tracker and must not be renamed:
  `[p2]` `[p3]` plan phases · `[b1]` `[b2]` `[b3]` backend blocks ·
  `[prep:react]` `[prep:next]` `[prep:ts]` `[prep:qa]` `[prep:sec]` `[prep:css]` `[prep:arch]` `[prep:scen]` interview-prep areas.
- Frontend comes first. Backend is deliberately "essentials only". Anything in **Later** stays parked until after the offer.

## v1 scope

**In:** sign in · decks · cards (front, back, example) · daily review with spaced repetition ·
optimistic grading · basic stats · daily "cards due" email · deployed with a real domain.

**Out of v1:** Redis, AI features, file uploads, admin, sharing decks, mobile app. See **Later**.

---

## Phase 0 — Skeleton, deployed on day one `[p2]` `[b1]`

**Build**
- [ ] One evening in `playground/`: raw `node:http` server, zero framework — read the body stream, parse JSON, route, status codes, graceful shutdown
- [ ] npm workspaces: `apps/web`, `apps/api`, `packages/shared`
- [ ] `docker-compose.yml` with Postgres 17; `.env.example` documenting every variable
- [ ] Fastify on Node 24 with `node --watch`; `tsc --noEmit` for type checks; `GET /health`
- [ ] Next.js 16 app that renders data from the API on the server
- [ ] Deploy now: web on Vercel, API + Postgres on Railway. Ship the skeleton, then grow it

**Done when**
- [ ] Wrote a working HTTP server with zero dependencies, and can explain what Fastify adds on top
- [ ] Fresh clone runs with `docker compose up -d` plus one command per app
- [ ] A public URL shows data that went browser → Next server → API → Postgres and back

**Interview story:** "What does a Node framework do under the hood?" — answered from the raw server.

---

## Phase 1 — Decks and cards, end to end `[p2]` `[b1]` `[b2]` `[prep:next]` `[prep:ts]`

**Build**
- [ ] Schema + migrations: `users`, `decks`, `cards`, `reviews`; `snake_case`, real foreign keys and constraints
- [ ] Raw SQL with `pg` for the first queries (at least 3 stay raw for good), then Drizzle
- [ ] Fastify CRUD for decks and cards: Zod validation, cursor pagination, one error shape `{ error: { code, message, details? } }`
- [ ] Zod schemas in `packages/shared`, used by both Server Actions and the API
- [ ] Integration tests (Vitest) against a running server and a real test database
- [ ] Graceful shutdown on `SIGTERM`: stop accepting, finish in-flight requests, drain the `pg` pool
- [ ] Next: reads in Server Components, mutations via Server Actions + `useActionState`
- [ ] Caching on purpose: what is cached, and how it's invalidated after each mutation
- [ ] Streaming with Suspense; `loading` / `error` / empty states for every route
- [ ] README section: the Server vs Client Component boundary and why it sits where it does

**Done when**
- [ ] API returns correct status codes and one consistent error shape
- [ ] Every input is validated before it reaches business logic
- [ ] Integration tests pass against a running server
- [ ] Migrations run forward and back cleanly
- [ ] Can explain CORS end to end: why Postman works and the browser doesn't
- [ ] No `any`, no `SELECT *`, no hardcoded URLs

**Interview story:** "Walk me through a mutation: from the form submit to the row in Postgres and back to a fresh UI."

---

## Phase 2 — Review flow and spaced repetition `[p2]` `[prep:react]` `[prep:qa]` `[prep:css]`

The product story, and most of the React interview material.

**Build**
- [ ] SM-2 scheduling as pure functions with unit tests (FSRS, used by modern Anki, as a stretch goal)
- [ ] API: due cards for today; submit a grade
- [ ] Review screen: flip animation with `transform` / `opacity` only
- [ ] Keyboard-first: Space flips, 1–4 grades; visible focus; screen-reader announcements
- [ ] Optimistic grading with `useOptimistic`: instant UI, rollback on failure
- [ ] Next card prefetched, so a session never waits on the network
- [ ] Review states as a discriminated union: `idle | loading | reviewing | done | error`
- [ ] Stats: due today, streak, reviews per day
- [ ] Component tests with Vitest + RTL (+ MSW for the network)

**Done when**
- [ ] A full review session works with the keyboard only
- [ ] Grading still feels instant on a throttled network
- [ ] I use the app daily for my own interview cards

**Interview story:** "Optimistic UI: what the user sees, what happens when the server says no, and how I tested it."

---

## Phase 3 — Auth `[p2]` `[b3]` `[prep:sec]`

**Build**
- [ ] Session auth by hand: argon2 hashing, session table, `HttpOnly` / `Secure` / `SameSite` cookies
- [ ] `proxy.ts` for redirects only (UX); a real check inside every Server Action and every API route
- [ ] Every query scoped to the current user; one user can never read or change another's cards
- [ ] Decide, and write down, whether to swap in a library (e.g. Better Auth) and why
- [ ] Playwright flow: sign in → review a deck

**Done when**
- [ ] Cross-user access is covered by a test that fails if a check is removed
- [ ] Can explain CSRF and why `SameSite` cookies matter
- [ ] Can explain why `proxy.ts` is not the security boundary

**Interview story:** "Authentication vs authorization in this app, and where each check lives."

---

## Phase 4 — Postgres where it shows in interviews `[b2]`

**Build**
- [ ] Seed script: 100k+ cards and reviews for realistic numbers
- [ ] **Slow query:** due-cards query → `EXPLAIN ANALYZE` → composite index `(user_id, due_at)` → remeasure
- [ ] **Concurrency:** the same card graded from two tabs at once → lost update. Reproduce it, then fix with a conditional `UPDATE`, `SELECT … FOR UPDATE`, or a unique constraint, and compare
- [ ] **N+1:** decks list with card counts → spot it in query logs → one query

**Done when**
- [ ] Before/after numbers for the index fix are recorded in **Measurements**
- [ ] Can explain transactions, isolation levels and connection pooling at a whiteboard level

**Interview story:** "Two tabs, one card: the lost update and the three ways I could fix it."

---

## Phase 5 — Production `[b3]` `[p2]` `[prep:scen]`

**Build**
- [ ] One background job: daily "cards due" email, with retries and backoff
- [ ] Sentry on frontend and backend
- [ ] CI: tests run before every deploy; push to main deploys
- [ ] Real domain
- [ ] Core Web Vitals / Lighthouse before and after one optimization
- [ ] Optional: Stripe "Pro" plan with a webhook that survives being delivered twice

**Breakage drill — pick three**
- [ ] Slow unindexed query → find the latency spike
- [ ] N+1 → spot the exploding query count
- [ ] Hanging downstream API → why you needed a timeout

**Done when**
- [ ] Errors reach Sentry with enough context to debug them
- [ ] Each breakage told as a 2-minute incident story
- [ ] Someone other than me has used it

**Interview story:** "Production got slow — here's how I found out why."

---

## Phase 6 — Polish `[p3]` `[prep:arch]`

- [ ] README: how to run it, architecture diagram, decisions, trade-offs, what's next
- [ ] 2-minute demo pitch, rehearsed in English
- [ ] Design flash_cards for 1M users, aloud
- [ ] Can explain every architectural choice in 2 minutes

---

## Later — after the offer

Redis caching and rate limiting in depth · AI features (example sentences over SSE, pgvector) ·
file uploads to S3/R2 · admin and shared decks · AWS · Go.

## Decisions log

Date, decision, why, alternatives considered.

## Measurements

Before/after numbers from experiments: index fix, Core Web Vitals, review latency.
