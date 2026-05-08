+++
date = '2026-05-08T04:24:51-07:00'
draft = false
description = 'A practical tutorial for producing Product Requirements Documents for large-scale web projects when AI agents are part of the team'
title = 'Prd Writing Guide'
categories = ['vibe-coding']
tags = ['AI coding', 'product requirements', 'documentation']
+++

A practical tutorial for producing Product Requirements Documents (PRDs) for large-scale web projects when AI coding agents (Claude Code, Cursor, Copilot, Codex) are part of the team.

## Table of Contents

1. [Why PRDs Are Different Now](#1-why-prds-are-different-now)
2. [Seven Best Practices](#2-seven-best-practices)
3. [The Sections of a Good PRD](#3-the-sections-of-a-good-prd)
4. [AI-Specific Patterns](#4-ai-specific-patterns)
5. [Lifecycle: Writing, Reviewing, Maintaining](#5-lifecycle-writing-reviewing-maintaining)
6. [Pre-Approval Checklist](#6-pre-approval-checklist)
7. [Templates](#7-templates)
8. [Common Mistakes](#8-common-mistakes)

---

## 1. Why PRDs Are Different Now

A traditional PRD is a human-to-human document. It assumes shared context — the reader knows the codebase, the team, the history. Gaps are filled by hallway conversations.

An AI-era PRD is read by **two audiences simultaneously**:

- **Humans** need _why_: motivation, tradeoffs, business context, history.
- **AI agents** need _what_: precise, unambiguous, machine-parseable specifications they can ground their code generation in.

The same paragraph that reads as "obvious" to a senior engineer can produce three subtly wrong implementations from an AI agent. AI tools fabricate when context is missing — they pattern-match from training data instead of project specifics. A vague PRD produces vague code.

This shifts what "good" looks like:

| Traditional PRD         | AI-era PRD                                  |
| ----------------------- | ------------------------------------------- |
| Prose-heavy             | Mix of prose + structured artifacts         |
| "Should be fast"        | "p95 ≤ 250ms, verified by `pnpm test:perf`" |
| Revision history        | Decision log with rationale                 |
| Stored in Confluence    | Stored in repo alongside code               |
| Updated when remembered | Updated as part of every PR                 |
| Goals, non-goals        | Goals, non-goals, **anti-requirements**     |
| Implicit conventions    | Explicit "AI operating manual"              |

---

## 2. Seven Best Practices

### BP1. Write for two readers simultaneously

Humans need narrative; AI needs structure. Mix both.

- Front-load **what** as machine-parseable artifacts (tables, code blocks, type definitions, ID'd requirements). Back-load **why** as prose.
- Use **RFC 2119 keywords** consistently: MUST, SHOULD, MAY, MUST NOT. AI tools weight these heavily.
- Distinguish **normative** content (binding) from **informative** content (illustrative). Mark them.
- **No pronouns across sections.** "It must validate the token" is fine inside one bullet but breaks when the AI loads only that section. Repeat the noun.

Add a legend at the top of every PRD:

```markdown
> **Normative content** is in tables, REQ-\* statements, and code blocks marked
> `contract`. **Informative content** (prose, examples, motivation) explains
> the why and is non-binding. The keywords MUST, SHOULD, MAY, MUST NOT are
> used per RFC 2119.
```

### BP2. Make every requirement executable

A non-executable requirement is a wish. An executable requirement has a verification command or test.

**Bad:**

> The system should be fast.

**Better:**

> P95 latency of `/api/orders` ≤ 600ms.

**Good:**

```markdown
### REQ-PERF-001 — Order creation latency

**Statement.** P95 latency of POST /api/orders MUST be ≤ 600ms under normal load.

**Verification.** `pnpm test:load` runs k6 script `tests/load/orders.js`,
asserts p95 ≤ 600ms over 5-min window at 50 RPS sustained.

**Rationale.** Matches current Spring gateway baseline (see Grafana board
"order-edge-p95"); regression breaks SLO commitment to checkout team.
```

Three things in one block: what, how to verify, why. AI agents now have a complete grounding triple.

### BP3. Layered specificity (zoom levels)

Long PRDs fail because readers lose the thread. Structure so any reader can extract the right layer:

| Layer                | Length      | Purpose                            | Audience                |
| -------------------- | ----------- | ---------------------------------- | ----------------------- |
| L0 — TL;DR           | 1 paragraph | What and why, in one breath        | Execs, AI system prompt |
| L1 — Goals/non-goals | 1 page      | Scope boundary                     | All readers             |
| L2 — User journeys   | 2–5 pages   | Behavior                           | PM, QA, designers       |
| L3 — Contracts       | 5–20 pages  | Schemas, endpoints, state machines | Engineers, AI agents    |
| L4 — Operations      | 2–5 pages   | Rollout, rollback, monitoring      | SRE, on-call            |
| L5 — Appendix        | unbounded   | Decision log, glossary, references | Future readers          |

**The L0 TL;DR is the single most valuable section.** AI tools with limited context budgets pin it as the snippet they remember. Write it last, after the rest of the doc has clarified your thinking.

### BP4. Co-locate with code; make AI find it

The PRD is worthless if the AI agent doesn't load it. Three mechanisms:

**Path conventions** AI tools index by default:

- `/docs/prd/*.md`
- `/specs/*.md`
- `/.cursor/rules/*.mdc` (Cursor)
- `CLAUDE.md` references (Claude Code)

**Bidirectional linking.** PRDs reference `CLAUDE.md`; `CLAUDE.md` references active PRDs. Example `CLAUDE.md` snippet:

```markdown
## Active PRDs

- [Next.js Migration](docs/prd/nextjs-migration.md) — load before any work in
  `frontend/`, `services/gateway-service/`, or auth-related areas.
- [Search Revamp](docs/prd/search-revamp.md) — load before work in
  `services/search-service/` or `app/(storefront)/search/`.
```

**Stable section anchors.** When the AI says "see PRD §7.4", that should still resolve in 6 months. Use ID-stable headings, not narrative ones (`## 7.4 Auth & session`, not `## Auth (after the recent rewrite)`).

### BP5. Decision log over revision history

Every "we chose X over Y" must be capturable, searchable, and dated. Without this, AI agents will happily undo your decisions because the prose framing was lost.

Replace "v1.2 — updated by Alice" with **ADRs** (Architecture Decision Records). One file per decision, in `docs/adr/`:

```markdown
---
id: ADR-0007
title: HttpOnly session cookie instead of bearer token in localStorage
status: Accepted
date: 2026-05-08
deciders: Backend lead, Security, Frontend lead
supersedes: []
superseded-by: []
---

## Context

Current SPA stores access token in JS memory + localStorage refresh token.
XSS risk and complexity in token rotation.

## Decision

Move to opaque session cookie (HttpOnly, Secure, SameSite=Lax). Server-side
session lookup. Bearer token never reaches the browser.

## Alternatives considered

- **JWE on cookie** — stateless edge, but harder to revoke. Rejected for
  revocation requirements (admin force-logout).
- **Auth.js / NextAuth** — second source of truth alongside identity-service.
  Rejected.

## Consequences

- Eliminates XSS token theft.
- One refresh path, server-locked.

* Edge becomes stateful (Redis lookup).
* Long-lived browser sessions on cutover need forced re-login.

## Verification

- DevTools shows no Bearer token in any storage.
- `Authorization` header absent from all browser→Next.js requests.
```

### BP6. Define non-goals AND anti-requirements

**Non-goal** = "we will not do X (for now)". Defers scope.
**Anti-requirement** = "we will not do X, ever, even if it seems obvious to add". Preempts AI over-engineering.

Anti-requirements are gold for AI agents because they catch the most common over-delivery patterns:

```markdown
### Anti-requirements (do not add even if tempted)

- **AR1**: Do NOT introduce a frontend ORM (Prisma, Drizzle). All data access
  goes through Route Handlers calling existing microservices.
- **AR2**: Do NOT add a Redux/Jotai store alongside Zustand. State libraries
  do not multiply.
- **AR3**: Do NOT add a "compatibility shim" for the old SPA's API client.
  Cutover is clean; no parallel API styles.
- **AR4**: Do NOT generate TypeScript types from upstream OpenAPI without
  explicit ADR; types stay hand-authored to keep contracts visible.
```

### BP7. Slice vertically, not horizontally

A horizontal slice ("build all the API routes, then all the UI") leaves nothing demonstrable until the end. A vertical slice ("login screen + login route handler + middleware decode") works end-to-end immediately.

Each milestone should:

1. Ship a thin, end-to-end working feature
2. Be independently demoable in 2 minutes
3. Be independently rollback-able
4. Be implementable by an AI agent in one or two focused sessions

Add a **demo script** per milestone — it doubles as an AI verification prompt:

```markdown
### M1 demo script

1. Visit staging Next.js
2. Click "Login with GitHub"
3. Authorize
4. Land on `/`, see header username
5. DevTools: confirm only `cf_session`, `cf_refresh`, `cf_device` cookies; no
   localStorage tokens
6. Wait 16 minutes (or force expiry)
7. Click any link → confirm silent refresh, no logout
8. Click logout → cookies cleared, redirected to `/`
```

---

## 3. The Sections of a Good PRD

Below is a complete table of contents with templates. Not every project needs every section, but the order matters: top-down readability is a feature.

### §0. TL;DR

One paragraph. What and why, in one breath. Write this last.

```markdown
## 0. TL;DR

Replace the SPA + Spring Cloud Gateway with a single Next.js (App Router) app
as the only public-facing edge. Microservices are unchanged. Access tokens
move out of the browser into HttpOnly cookies; rate limiting, CORS, JWT
validation, and tracing move from gateway-service into Next.js middleware
and Route Handlers. Phased over six milestones; each independently rollback-
able. Success = one public deployable, parity SLOs, no Bearer token in
browser memory.
```

### §1. Status & ownership

```markdown
| Field             | Value                                                           |
| ----------------- | --------------------------------------------------------------- |
| Status            | Draft / In-Review / Approved / In-Flight / Shipped / Superseded |
| Author            | name                                                            |
| Engineering owner | name (single point of contact)                                  |
| Reviewers         | [name + role] (each must approve before status → Approved)      |
| Stakeholders      | [name + interest] (informed, not blocking)                      |
| Target start      | YYYY-MM-DD                                                      |
| Target completion | YYYY-MM-DD                                                      |
| Tracking issue    | link                                                            |
| Related PRDs      | links                                                           |
| Supersedes        | links                                                           |
```

### §2. Context

Three paragraphs, no more:

1. **Current state** — factual, present tense.
2. **What changed externally** — why now? (regulation, customer ask, scale event, technical debt cliff)
3. **What we've already tried or considered** — sets up the decision log.

### §3. Problem statement

One sentence problem + one sentence solution direction. Test: if you remove this section, do readers still know what the project _is_?

### §4. Goals

Goals get an ID, a sentence, and a _measure_ — never a goal without a measure.

```markdown
| ID  | Goal                                     | Measure                                                                                       |
| --- | ---------------------------------------- | --------------------------------------------------------------------------------------------- |
| G1  | Replace SPA with Next.js, feature-parity | All routes (§6.1) reachable, all journeys pass E2E                                            |
| G3  | Move access tokens off the browser       | DevTools audit shows zero Bearer tokens in storage; zero `Authorization` headers from browser |
```

**Anti-pattern:** "G3: Improve security." Unmeasurable, unverifiable, useless.

### §5. Non-goals & anti-requirements

```markdown
## 5. Non-goals

- **NG1**: Microservice rewrites. Java/Spring stack stays as-is.
- **NG2**: Database schema changes.
- **NG3**: Server Actions for write paths in first cut.

## 5.1 Anti-requirements

- **AR1**: Do NOT add a frontend ORM.
- **AR2**: Do NOT introduce a second state library.
```

### §6. Users & scenarios

Three sub-sections:

**6.1 Personas** — who, with permission scopes:

```markdown
| Persona           | Roles      | Primary jobs                           |
| ----------------- | ---------- | -------------------------------------- |
| Anonymous visitor | none       | Browse catalog, register               |
| Customer          | ROLE_USER  | Browse, cart, checkout, manage orders  |
| Admin             | ROLE_ADMIN | Manage products, tickets, redeem codes |
```

**6.2 Routes / surfaces** — exhaustive table of public surfaces:

```markdown
| Route               | Auth         | Render mode               | Notes                  |
| ------------------- | ------------ | ------------------------- | ---------------------- |
| `/`                 | Anonymous OK | RSC + client islands      | Storefront home        |
| `/checkout/payment` | Required     | RSC + client              | Renders payment intent |
| `/admin/*`          | ROLE_ADMIN   | RSC layout, client tables | Admin shell            |
```

**6.3 User journeys with risk** — capture happy path AND unhappy paths:

```markdown
### J3 — Logged-in checkout via Alipay

**Happy path:**

1. User on `/checkout/payment` clicks "Pay with Alipay"
2. POST `/api/trade/orders` → returns `orderId`
3. POST `/api/finance/payments/alipay/intent` → returns redirect URL
4. Browser redirects to Alipay, user pays
5. Alipay async-notify hits `/api/payments/alipay/notify`
6. Browser returns to `/orders/payment-return?orderId=...`
7. UI polls `/api/trade/orders/:id` until `paid: true`

**Unhappy paths:**

- 3a. Stock depleted between 2 and 3 → 409, redirect to `/cart`
- 5a. Async-notify fails signature → 400, finance-service does not mark paid
- 5b. Async-notify arrives before browser return → UI shows "paid" on first poll
- 6a. User closes browser before return → next visit to `/orders` shows correct state
```

### §7. Architecture

- **Topology diagrams before/after** (ASCII art is fine and AI-readable)
- **What's inherited / what isn't** for migrations
- **Component-level deep dives** for load-bearing pieces

For each load-bearing component, include:

1. Inputs / outputs (interface)
2. State (what does it own?)
3. Failure modes (what breaks, and how do we know?)
4. Concurrency model (single-flight? lock? lease?)

### §8. API contracts

For each endpoint, the contract block:

````markdown
#### POST /api/auth/login

**Auth:** none required (this _establishes_ the session)

**Request**

```ts
type LoginRequest = {
  identifier: string; // email or username
  password: string;
  deviceId?: string; // optional override; defaults to cf_device cookie
};
```

**Response 200**

```ts
type LoginResponse = {
  user: { id: string; username: string; roles: string[] };
  // no token in body — set via Set-Cookie: cf_session, cf_refresh
};
```

**Errors**
| Status | Code | When |
|---|---|---|
| 400 | INVALID_INPUT | identifier/password missing or malformed |
| 401 | INVALID_CREDENTIALS | wrong password (generic — do not leak which) |
| 423 | ACCOUNT_LOCKED | too many failures |
| 429 | RATE_LIMITED | per IP, 5 attempts / 5 min |

**Side effects**

- Sets cookies `cf_session`, `cf_refresh`, `cf_device` (if absent)
- Emits log event `auth.login` with `userId`, `result`
- Increments metric `auth_login_total{result}`

**Upstream call**

- POST identity-service `/auth/login` with `{ identifier, password, deviceId }`
- Internal token: none (login is unauthenticated upstream too)
````

A handler with this contract is implementable by an AI agent in one shot.

### §9. Data model

- ER diagram or table
- **Schema definitions as code** (TypeScript / Prisma / SQL) — not prose
- Data lifecycle: creation, update, soft-delete, retention
- PII classification & compliance fields

```ts
// contract: Order
type Order = {
  id: string; // UUID v4
  userId: string;
  status: "PENDING" | "PAID" | "FULFILLED" | "CANCELLED" | "REFUNDED";
  items: OrderItem[];
  totalCents: number; // never floating point for money
  currency: "CNY" | "USD";
  createdAt: string; // ISO 8601
  paidAt: string | null;
  fulfilledAt: string | null;
};
```

### §10. UI/UX specifications

- Design system reference (tokens, components used)
- Wireframes or Figma links per screen
- Responsive breakpoints
- Accessibility (WCAG level, keyboard, ARIA)
- i18n / l10n scope

For each screen, document **all states**: empty, loading, partial, error, success.

### §11. Non-functional requirements

Categories to always cover:

- **Latency** — P50, P95, P99 (never just "fast")
- **Throughput** — RPS sustained, peak, duration
- **Availability** — SLO % + tolerance for AZ/region failure
- **Capacity** — concurrent users, DB connections, memory ceiling
- **Security** — auth, secrets, threat model coverage
- **Privacy** — PII handling, retention, redaction
- **Compatibility** — browsers, OS, screen sizes
- **Accessibility** — WCAG level, keyboard, screen reader, contrast
- **i18n / l10n**
- **Build / CI** — build time, bundle budgets
- **Cost** — per-RPS infra cost ceiling (often forgotten)

```markdown
| ID     | Category   | Requirement                                                   |
| ------ | ---------- | ------------------------------------------------------------- |
| NFR-1  | Latency    | P50 of `/api/auth/me` ≤ 80ms, P95 ≤ 250ms                     |
| NFR-3  | Throughput | 200 RPS sustained, 500 RPS peak for 30s                       |
| NFR-10 | Build      | `next build` ≤ 5 min on CI; initial JS ≤ 250KB gzipped on `/` |
```

### §12. Observability

The pattern: every requirement says **what to log/measure**, **what threshold triggers an alert**, **who gets paged**.

```markdown
> **REQ-OBS-005**: SLO burn-rate alert. P95 latency on `/api/trade/orders`
> exceeding 600ms (NFR-2) for 5 minutes triggers a warning page to
> `#oncall-trade`; for 30 minutes triggers a critical page to PagerDuty
> rotation `trade-edge`.
```

Cover:

- Structured log fields (with redaction rules)
- Metric names and labels
- Dashboards (which boards exist, what panels)
- Alerts (threshold, window, severity, recipient)

### §13. Risks

```markdown
| ID  | Risk                     | Likelihood | Impact   | Mitigation                      | Owner     | Status |
| --- | ------------------------ | ---------- | -------- | ------------------------------- | --------- | ------ |
| R1  | SSE proxy buffers        | Med        | High     | Prototype + load test before M3 | @ai-team  | Open   |
| R2  | Alipay signature regress | Med        | Critical | Replay 100 captured payloads    | @payments | Open   |
```

Run a **pre-mortem** before approval: a 30-minute "imagine it failed — why?" session. Add the top 3 unspoken risks.

### §14. Test plan

**Pyramid + per-milestone gates.** A milestone is not "done" until its gate is green.

```markdown
### Per-milestone gates

- M1: J5 E2E green; auth integration suite green; cookie audit clean
- M2: J1 E2E green; bundle budget green
- M3: J2/J3/J4 E2E green; Alipay replay 100/100; load test 200 RPS green
- M4: J6/J7/J8 E2E green; SSE 30s drop test green
- M5: rate limit acceptance green; security scan clean; full E2E green
- M6: cutover smoke green; 72h monitoring green
```

Cover: unit, integration, E2E, load, security, cutover smoke.

### §15. Migration / rollout plan

Each milestone: **scope, acceptance, rollback, demo script.**

```markdown
### M3 — Checkout, payment, fulfillment (3 weeks)

**Scope.** Migrate `/checkout/*`, `/orders/*`, `/wallet/recharge*`. Implement
proxies for `/api/trade/**`, `/api/finance/**`, `/api/fulfillment/**`.
Implement `/api/payments/alipay/notify`.

**Acceptance.** J2, J3, J4 pass in staging. Async-notify replay 100/100.

**Rollback.** Toggle `EDGE=spa` at load balancer; SPA + gateway resume serving.

**Demo script.**

1. Add item to cart, checkout via balance → order paid + fulfilled
2. Add item to cart, checkout via Alipay → redirect, pay, return → order paid
3. Wallet recharge via Alipay → balance increased
4. Verify all three orders visible at `/orders` with correct state
```

### §16. Open questions

Every open question **owned** and **timed**:

```markdown
| ID  | Question                                     | Owner     | Decide by  | Decision |
| --- | -------------------------------------------- | --------- | ---------- | -------- |
| Q1  | Hosting target — self-host, Vercel, other?   | @platform | 2026-05-22 | TBD      |
| Q2  | Rate-limit library — Upstash vs custom Redis | @backend  | 2026-06-15 | TBD      |
```

When decided, update the row with the answer and link to the ADR.

### §17. Glossary

For domain terms, define **what it is, what it isn't, what it's often confused with**:

```markdown
- **Order paid**: trade-service has marked `payment_status = PAID`.
  - Not the same as: order fulfilled (fulfillment-service has issued codes).
  - Not the same as: order delivered (user notified).
  - Source of truth: trade-service `orders.payment_status` column.
```

### §18. Appendix

- **18.1 File-level pointers** — paths to authoritative files in the current codebase. Critical for AI grounding.
- **18.2 Decision log seed** — list of decisions to promote to ADRs.
- **18.3 Out-of-band assets** — DNS names, certs, third-party config required before kickoff.
- **18.4 Fixtures** — captured payloads (OAuth callbacks, payment notifies, SSE chunks) for replay tests.

### §19. AI agent operating manual

Dedicated section telling AI agents _how_ to do the work, not just what:

```markdown
## 19. AI agent operating manual

### Context loading

When working on this migration, AI agents MUST load:

1. This PRD
2. CLAUDE.md (root)
3. Relevant §18.1 file pointers for the area being changed

### Code conventions

- Route Handlers live under `app/api/<service>/[...path]/route.ts`
- Each handler exports named HTTP methods (`GET`, `POST`, …)
- Upstream calls go through `lib/upstream.ts` — never `fetch()` directly
- Errors map to `lib/errors.ts` taxonomy — no ad-hoc `Response.json({error:...})`

### Forbidden

- Do not import from `frontend/src/**` — that tree is being deleted
- Do not add `next-auth`, `iron-session`, or other session libraries
- Do not call `localStorage` or `sessionStorage` for auth-related data
- Do not use `'use server'` server actions for write paths

### Verification commands

- Type check: `pnpm typecheck`
- Unit: `pnpm test`
- Integration: `pnpm test:integration`
- E2E: `pnpm test:e2e`
- Cookie audit: `pnpm audit:cookies`
- Bundle budget: `pnpm build && pnpm size-limit`

### Done definition for any change

1. All verification commands green
2. Affected REQ-\* IDs listed in the PR description
3. New REQ-\* added to PRD if behavior was specified that wasn't there

### Common AI mistakes — read this

- Next.js App Router does NOT need `next/router` — use `next/navigation`
- Route Handlers run on Node by default; Edge runtime is FORBIDDEN for SSE
- Cookies are set via `NextResponse.cookies.set()`, not raw `Set-Cookie`
- `cf_session` is HttpOnly — `document.cookie` cannot read it
- The project does NOT use Auth.js / NextAuth — do not import it
- The project uses pnpm — do not run `npm install` or `yarn`
```

---

## 4. AI-Specific Patterns

### Pattern 1: Traceability matrix

A table linking REQ → test → code:

```markdown
| REQ          | Test                              | Implementation   | Status      |
| ------------ | --------------------------------- | ---------------- | ----------- |
| REQ-AUTH-001 | tests/audit/cookies.spec.ts       | lib/session.ts   | Implemented |
| REQ-AUTH-003 | tests/integration/refresh.spec.ts | middleware.ts:42 | In progress |
| REQ-EDGE-001 | tests/load/trade-orders.js        | lib/rateLimit.ts | Pending     |
```

The #1 thing AI agents lack is knowing _which test verifies which requirement_. Hand-maintain this matrix; it pays back tenfold.

### Pattern 2: Captured-fixture appendix

For payment notifies, OAuth callbacks, SSE chunks — capture real payloads:

```markdown
### 18.4 Fixtures

- `tests/fixtures/alipay-notify-paid.json` — successful payment notify
- `tests/fixtures/alipay-notify-refund.json` — refund notify
- `tests/fixtures/oauth-github-success.json` — GitHub OAuth callback
- `tests/fixtures/sse-chat-stream.txt` — 30s captured AI chat stream

Refresh via: `pnpm capture:fixtures --env staging`
```

These fixtures are the input to integration tests. AI agents can implement against fixtures without needing live services.

### Pattern 3: Hallucination guards

A short list of facts the AI tends to get wrong about your project, restated explicitly. See §19 above. This catches AI agents who pattern-match from training-data prevalence rather than your specifics.

### Pattern 4: Demo script as AI prompt

A demo script doubles as a verification prompt: "run the M1 demo script using Playwright; report any step that fails." If the script is precise enough for a human QA, it's precise enough for an AI agent.

### Pattern 5: Stable IDs for everything

Every requirement, goal, risk, journey, route, and milestone gets a stable ID:

- Goals: `G1`, `G2`, …
- Non-goals: `NG1`, `NG2`, …
- Anti-requirements: `AR1`, `AR2`, …
- Requirements: `REQ-AUTH-001`, `REQ-EDGE-001`, …
- Journeys: `J1`, `J2`, …
- Risks: `R1`, `R2`, …
- Open questions: `Q1`, `Q2`, …
- Non-functional: `NFR-1`, `NFR-2`, …
- Milestones: `M1`, `M2`, …

Stable IDs survive rewrites and let AI agents reference exactly. PR descriptions can say "implements REQ-AUTH-003 and REQ-AUTH-005, mitigates R5" — instantly searchable, instantly verifiable.

---

## 5. Lifecycle: Writing, Reviewing, Maintaining

### Writing

1. **Outline first, in 30 minutes.** Just headings, no content. Get sign-off on shape before drafting.
2. **Draft top-down.** TL;DR → goals → architecture → contracts → ops. Don't start with the API table.
3. **Draft with the AI in the loop.** Ask the AI to identify ambiguities — it's an excellent ambiguity detector. Paste a section and ask "what could be interpreted multiple ways here?"
4. **Examples first, prose second.** When stuck, write the example payload, then the prose explaining it.
5. **TL;DR last.** You don't know what the doc says until you've finished it.

### Reviewing

Use the [Pre-Approval Checklist](#6-pre-approval-checklist) below. Reject PRDs that don't meet it; reviewing a half-baked PRD wastes everyone's time.

Specific reviewer assignments:

- **Frontend lead** — UI/UX, journeys, routes
- **Backend lead** — API contracts, data model, architecture
- **Platform/SRE** — observability, NFRs, rollout, rollback
- **Security** — auth, threat model, NFRs related to security
- **Product** — goals, non-goals, success metrics

### Maintaining

A PRD that doesn't get updated becomes a lying document — the worst kind. Worse than no PRD, because AI agents will trust the lies.

**Rules:**

1. **PR template includes** "PRD updated? Y/N + REQ-\* IDs touched"
2. **Status field ratchets** Draft → Approved → In-Flight → Shipped → Superseded; never goes backward without a comment
3. **After Shipped, freeze and archive.** Link to its successor.
4. **Open questions resolved within milestone** get promoted to ADRs and removed from §16
5. **`last_updated` is real**, updated on every meaningful edit
6. **Quarterly audit.** Walk every PRD; archive shipped ones; flag stale ones.

### When to split a PRD

A PRD over ~500 lines is a smell. Split when:

- Two sections could ship independently → two PRDs with shared context doc
- Ownership splits across teams → each team owns its PRD
- A section is reused across PRDs → extract to a standalone doc, reference from multiple PRDs

Common extraction candidates:

- Auth/session design → `docs/auth/session-design.md`
- Streaming architecture → `docs/architecture/sse-streaming.md`
- Glossary → `docs/glossary.md`
- ADRs → `docs/adr/ADR-XXXX.md`

---

## 6. Pre-Approval Checklist

Use this before moving status from Draft to Approved:

- [ ] §0 TL;DR is exactly one paragraph
- [ ] Every goal has a measure
- [ ] Every non-goal is concrete and non-aspirational
- [ ] §5.1 anti-requirements section exists
- [ ] Every REQ-\* is atomic, ID'd, and testable
- [ ] Every endpoint in §8 has request/response/error schemas
- [ ] Every NFR has a number, not an adjective
- [ ] Every risk has owner + mitigation + status
- [ ] Every open question has owner + decide-by date
- [ ] §19 AI operating manual section exists
- [ ] Glossary distinguishes commonly-confused terms
- [ ] §18.1 file-level pointers list authoritative current-state files
- [ ] Demo script per milestone in §15
- [ ] Verification command per REQ (or batched in §14)
- [ ] Pre-mortem held; top risks captured
- [ ] All required reviewers listed in §1 and have approved
- [ ] PRD is committed in repo at a stable path
- [ ] `CLAUDE.md` (or equivalent) references the PRD

---

## 7. Templates

### Minimal PRD skeleton

Copy this when starting a new PRD:

```markdown
---
title: <Project name>
status: Draft
owner: TBD
last_updated: YYYY-MM-DD
related: []
---

# <Project name> — PRD

## 0. TL;DR

<One paragraph. Write last.>

## 1. Status & ownership

| Field             | Value |
| ----------------- | ----- |
| Status            | Draft |
| Author            |       |
| Engineering owner |       |
| Reviewers         |       |
| Target start      |       |
| Target completion |       |
| Tracking issue    |       |

## 2. Context

## 3. Problem statement

## 4. Goals

## 5. Non-goals

### 5.1 Anti-requirements

## 6. Users & scenarios

### 6.1 Personas

### 6.2 Routes / surfaces

### 6.3 User journeys

## 7. Architecture

## 8. API contracts

## 9. Data model

## 10. UI/UX

## 11. Non-functional requirements

## 12. Observability

## 13. Risks

## 14. Test plan

## 15. Migration / rollout plan

## 16. Open questions

## 17. Glossary

## 18. Appendix

### 18.1 File-level pointers

### 18.2 Decision log seed

### 18.3 Out-of-band assets

### 18.4 Fixtures

## 19. AI agent operating manual
```

### Requirement template

```markdown
### REQ-<AREA>-<NNN> — <One-line title>

**Statement.** <Single sentence with MUST / SHOULD / MAY.>

**Verification.** <Command to run, file path of test, or audit procedure.>

**Rationale.** <Why this requirement exists; what breaks if it's wrong.>
```

### ADR template

```markdown
---
id: ADR-NNNN
title: <Decision in one line>
status: Proposed | Accepted | Deprecated | Superseded
date: YYYY-MM-DD
deciders: [names]
supersedes: []
superseded-by: []
---

## Context

## Decision

## Alternatives considered

## Consequences

## Verification
```

### Endpoint contract template

````markdown
#### <METHOD> <path>

**Auth:** <none | session | role:ADMIN | …>

**Request**

```ts
type RequestT = {
  /* … */
};
```

**Response 200**

```ts
type ResponseT = {
  /* … */
};
```

**Errors**
| Status | Code | When |
|---|---|---|

**Side effects**

- <cookies, logs, metrics, queue events…>

**Upstream call**

- <method + path + auth>
````

---

## 8. Common Mistakes

### "We'll figure it out as we go"

PRDs without contracts force every implementer (human or AI) to invent. AI agents will invent confidently and inconsistently. **Specify the contract before writing the code, not after.**

### Goals without measures

"Improve performance." Improve from what to what, measured how? An AI agent reading this writes plausibly-fast code that doesn't necessarily move the metric. Always: number, threshold, verification command.

### Hidden assumptions

If a senior engineer would "just know" something, write it down. AI agents don't have hallway conversations. The phrase "obviously we'd …" is a signal you have a hidden assumption — surface it.

### Prose-only specifications

A paragraph describing a JSON shape is harder to ground than the JSON shape itself. When in doubt, show the artifact.

### One PRD to rule them all

A 2000-line PRD is unreviewable, unmaintained, and lies within months. Split aggressively. Reuse shared design docs. PRDs are scoped to _projects_, not products.

### Stale PRDs left in the repo

If a PRD is shipped or abandoned, archive it. Move to `docs/prd/archive/` with status updated. Otherwise AI agents will keep loading it as if it's current.

### "We'll write the tests later"

If the test isn't specified, the requirement isn't real. AI agents will write code that compiles and looks plausible — but without tests as anchors, you can't know if it actually meets the requirement. Specify verification _with_ the requirement.

### No anti-requirements

You forgot to say "do NOT add an ORM" — so the AI added an ORM. The default behavior of AI agents is over-delivery. Anti-requirements are the brakes.

### Skipping the AI operating manual

You assumed the AI would "just figure out" your conventions. Three different agents make three different choices, and your codebase fragments. The AI operating manual is cheap insurance.

### No demo script

You said "M3 done when checkout works" — but works _how_? Nobody can verify. A demo script is a 2-minute scripted walkthrough; if you can't write one, the milestone isn't well-defined.

---

## Further reading

- [RFC 2119 — Key words for use in RFCs to Indicate Requirement Levels](https://www.rfc-editor.org/rfc/rfc2119)
- [ADR GitHub organization — examples and tooling](https://adr.github.io/)
- [Google's "Design Docs at Google" essay](https://www.industrialempathy.com/posts/design-docs-at-google/)
- [Anthropic's Claude Code documentation on `CLAUDE.md`](https://docs.claude.com/en/docs/claude-code/memory)

---

_This is a living document. Improvements welcome — open a PR._
