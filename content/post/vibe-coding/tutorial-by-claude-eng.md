+++
date = '2026-05-06T10:50:55+08:00'
title = "Tutorial: Building a Full-Stack Web Project with Claude Code"
tags = ['vibe-coding', 'coding-agent']
categories = ['vibe-coding']
+++

# Building a Full-Stack Web Project with Claude Code

A concrete, session-by-session walk-through of using Claude Code as a co-developer on a real, multi-module web application — designed around context-window constraints rather than against them.

The example project is **FocusBoard**, a team kanban with time tracking and reports. It's big enough to demonstrate the workflow but small enough to be specific.

---

## The Project

**FocusBoard** lets small teams plan work on kanban boards, track time per card, and see weekly reports.

### Features (in build order)

1. Auth (email + password, sessions)
2. Workspaces & memberships (invite by email)
3. Boards & columns
4. Cards (CRUD, drag/drop, assignees, labels, due dates)
5. Comments + activity log
6. Time tracking (start/stop timer per card)
7. Real-time updates (across users in a workspace)
8. Reports (per-user / per-workspace weekly view)
9. Settings, billing-ready hooks, deploy

### Stack

- pnpm + Turborepo monorepo
- Next.js 15 (App Router), React Server Components
- tRPC for the API (end-to-end type safety = Claude's best friend)
- Drizzle ORM + PostgreSQL
- Auth.js (NextAuth v5)
- Tailwind + shadcn/ui
- Pusher Channels for real-time
- Vitest (unit/integration) + Playwright (e2e)

**Why this stack for a Claude-driven build:** every layer is type-safe end-to-end. The TypeScript compiler is your most reliable code reviewer between sessions.

---

## Repo Layout

```
focusboard/
├── apps/
│   └── web/                    # Next.js app (UI + API routes)
├── packages/
│   ├── db/                     # Drizzle schema, migrations, queries
│   ├── api/                    # tRPC routers (the contract layer)
│   ├── ui/                     # shadcn components + custom
│   └── shared/                 # Zod schemas, enums, utility types
├── docs/
│   ├── ARCHITECTURE.md         # System overview (~1 page)
│   ├── DECISIONS.md            # ADR log
│   ├── CONVENTIONS.md          # Naming, error handling, patterns
│   └── features/               # One file per feature
│       ├── auth.md
│       ├── boards.md
│       └── ...
├── CLAUDE.md                   # Root instructions for Claude
├── .github/workflows/ci.yml
└── turbo.json
```

---

## Phase 0 — Foundations (before any feature)

**Goal:** Set up infrastructure so every later session starts on solid ground. Do this manually (or with `create-turbo`); don't burn a Claude session on boilerplate it can't run anyway.

### 0.1 Initial Claude session: bootstrap CLAUDE.md

After scaffolding the monorepo, run a single session:

```
You're seeing FocusBoard, a brand-new monorepo. Read package.json,
turbo.json, the tsconfig files, and the directory structure. Then
write CLAUDE.md at the root.

Include:
- Stack summary (one paragraph)
- Common commands (pnpm dev, pnpm test, pnpm db:migrate)
- Where things live (apps/, packages/, docs/)
- Hard rules: "always run pnpm typecheck after backend changes",
  "never edit packages/db/migrations — generate via pnpm db:generate"
- Pointer: "Before working on a feature, read docs/features/<name>.md
  if it exists, plus docs/CONVENTIONS.md."

Keep it under 80 lines. No fluff.
```

Then hand-write the strategic docs yourself (Claude can help draft, but the _content_ must come from you):

**`docs/ARCHITECTURE.md`** (~1 page):

```
# FocusBoard Architecture

## Data flow
Browser → Next.js Server Action / tRPC → Drizzle → Postgres
                                       ↘ Pusher → other browsers

## Auth boundary
Auth.js session middleware runs in apps/web/middleware.ts.
Every tRPC procedure resolves `ctx.user` or throws UNAUTHORIZED.

## Module dependencies
ui      → shared
api     → db, shared
db      → shared
web     → api, ui, db (read-only types)

ESLint enforces: ui MUST NOT import from db or api.
```

**`docs/DECISIONS.md`** starts as one entry:

```
## ADR-001: tRPC over REST  (2026-01-15)
Chose tRPC because end-to-end types make Claude-driven changes safer.
Trade-off: harder to consume from non-TS clients. Acceptable — no
mobile/3rd-party plans for v1.
```

**`docs/CONVENTIONS.md`** captures the patterns that should be uniform across the codebase: error shapes, Zod-first input validation, file naming, "every mutation logs to the activity table," etc.

### 0.2 CI as a forcing function

```yaml
# .github/workflows/ci.yml
- run: pnpm typecheck
- run: pnpm lint
- run: pnpm test
- run: pnpm build
```

This is non-negotiable. It's your asynchronous code reviewer that catches Claude's drift between sessions.

---

## Phase 1 — Auth (Session 1, ~2 hours)

**Why this first:** every other feature depends on `ctx.user`. Build the smallest thing that works end-to-end.

### Session 1 prompt

```
We're building auth for FocusBoard. Read:
- CLAUDE.md
- docs/ARCHITECTURE.md
- docs/CONVENTIONS.md
- packages/db/src/schema.ts (currently empty)
- apps/web/src/app/layout.tsx

Then propose a plan in plan mode. Requirements:
- Email + password signup/login
- Auth.js v5 with credentials provider
- Drizzle adapter
- Zod validation on all inputs
- Session via JWT (cookie)
- A /login page and /signup page using shadcn forms
- Tests: at minimum, signup → login → access /app/dashboard

Stop at the plan. Don't write code yet.
```

Review the plan. Push back where needed. Then:

```
Approved. Implement. After each file, run pnpm typecheck.
Update docs/features/auth.md as you go — schema, routes, edge
cases, and one paragraph of "what to know if you ever touch this."
```

### Context budget for this session (~200K window)

| What                                    | Tokens   |
| --------------------------------------- | -------- |
| System + tools                          | ~12K     |
| CLAUDE.md + docs auto-loaded            | ~5K      |
| Schema, layout, auth.js docs (WebFetch) | ~15K     |
| Plan + iteration                        | ~10K     |
| Generated code + edits                  | ~30K     |
| **Total used**                          | **~70K** |

Plenty of headroom. **End the session with a commit and `/clear`** before moving on.

### Output of this session

- `packages/db/src/schema/users.ts` — users, sessions, accounts tables
- `apps/web/src/lib/auth.ts` — NextAuth config
- `apps/web/src/app/(auth)/{login,signup}/page.tsx`
- `apps/web/src/middleware.ts`
- `docs/features/auth.md` — **the most important artifact for future sessions**
- A passing Playwright test

---

## Phase 2 — Workspaces (Session 2)

```
Read docs/features/auth.md and packages/db/src/schema/users.ts.
We're adding workspaces. Each user can create a workspace and
invite others by email.

Design (read first, don't code):
- packages/db/src/schema/workspaces.ts
- packages/api/src/routers/workspaces.ts (tRPC)
- packages/shared/src/validators/workspace.ts (Zod)

Constraints from CONVENTIONS.md still apply. Use procedure types
already established in api/src/trpc.ts (auth, public).

Plan first.
```

Notice: Claude only reads what's relevant. We don't load the entire `apps/web/` UI tree — workspace creation UI is a separate sub-task.

After backend lands, a second sub-session for UI:

```
Read docs/features/auth.md, docs/features/workspaces.md, and
apps/web/src/components/forms/ (the existing form patterns).
Build the /app/new-workspace page using the same form pattern as
signup. Wire to api.workspaces.create.
```

---

## Phase 3 — Boards & Cards (the meaty feature, 3 sessions)

This is where most large-project pain shows up. Done wrong, you'll have one giant 4000-line session that runs out of context and produces garbage. Done right, three clean sessions of ~1.5 hours each.

### 3.1 Contract session

**This is the most important session in the whole project.** Get it right and the next two are mechanical.

```
Read docs/ARCHITECTURE.md, docs/CONVENTIONS.md, and
docs/features/workspaces.md.

Design the data model and tRPC contract for boards. Do NOT
implement. Produce three files:

1. packages/db/src/schema/boards.ts (boards, columns, cards tables)
2. packages/shared/src/validators/board.ts (Zod input schemas)
3. packages/api/src/routers/boards.ts — but ONLY the procedure
   signatures with `.query()`/`.mutation()` and `// TODO` bodies.

Then write docs/features/boards.md describing:
- The data model
- Each procedure's contract (inputs, outputs, errors, auth rules)
- Open questions

Constraints:
- Boards belong to a workspace
- Columns are ordered (use fractional indexing — establish a
  helper in packages/shared/src/lib/ordering.ts if missing)
- Cards belong to a column, also fractionally ordered
- Soft delete only (deletedAt column)
```

The output is a _contract_. It's small (maybe 300 lines) and readable. The next two sessions implement it without needing to redesign anything.

### 3.2 Backend session

```
Read docs/features/boards.md and the three files from the
contract session. Implement the tRPC procedures. Write Vitest
tests for each — include auth checks and ordering correctness.

Don't touch the frontend. Don't change the schema unless the
contract is impossible (and if so, stop and explain).
```

This session has _exactly_ the context it needs: the contract + the implementation files. ~40-60K tokens used.

### 3.3 Frontend session

```
Read:
- docs/features/boards.md
- packages/api/src/routers/boards.ts (already implemented)
- apps/web/src/app/(app)/dashboard/page.tsx (existing pattern)
- apps/web/src/components/ui/ (shadcn primitives — list only, don't expand)

Build the board view at /app/w/[workspaceId]/b/[boardId].
Use @dnd-kit for drag/drop. Optimistic updates via tRPC's
useMutation onMutate.

Stop after the happy path works. We'll add polish in a follow-up.
```

The frontend session **never touches the database or tRPC implementation** — it only consumes the typed client. If the user drags a card and the API throws, that's a separate bug, fixed in a separate session.

---

## Phase 4 — Cross-cutting Feature: Real-time (Session 7)

Real-time touches every previous feature. This is where developers panic-load the entire codebase. Don't.

### Strategy: an integration ledger

Add to `docs/features/realtime.md` _before_ writing code:

```
# Real-time integration

Pusher channel per workspace: `workspace-${id}`.

Mutations that broadcast (must call `broadcast()` helper):

| Procedure                  | Event           | Payload     |
|----------------------------|-----------------|-------------|
| boards.create              | board:created   | Board       |
| cards.move                 | card:moved      | {id, col}   |
| cards.update               | card:updated    | Card        |
| comments.create            | comment:added   | Comment     |
| timer.start / timer.stop   | timer:changed   | TimerEvent  |

Subscribers: useWorkspaceEvents() hook in apps/web/src/lib/realtime.
```

Now your session is small:

```
Read docs/features/realtime.md.
Step 1: Add the broadcast() helper in packages/api/src/lib/pusher.ts.
Step 2: Wire it into the procedures listed in the table above.
For each: read the procedure file, add the broadcast call, run tests.

Show me the diff before moving to step 2.
```

Each procedure-touch is small. The doc is the index Claude uses to find work, not the codebase itself.

---

## Phase 5 — Drift Maintenance (every ~5-10 sessions)

After several features, things drift. Run a fresh "audit" session:

```
Read CLAUDE.md, docs/CONVENTIONS.md, and docs/features/*.md.
Then list 10-20 files at random under packages/api/src/routers/.

For each, check: does it follow the conventions? Are docs current?
Report a punch list. Don't fix anything yet.
```

Then fix in a separate session (the audit's findings become the prompt). Keep audit and fix sessions separate so the audit's full output doesn't pollute the fix's context.

---

## Anatomy of a Single Session

Here's a concrete session for "add labels to cards" — a small, realistic feature:

**Pre-session checklist (yours):**

- [ ] Last session committed?
- [ ] `/clear` if continuing same chat
- [ ] Know which docs/files are relevant
- [ ] Have a one-paragraph goal in mind

**Prompt 1 — Orient:**

```
Read docs/features/cards.md and packages/db/src/schema/boards.ts.
We're adding labels: each card has 0..N labels. Labels belong to
a board (not a workspace). Color + name. Only board members can
manage labels.

What's the smallest schema change? Plan only.
```

→ Claude proposes a `labels` table and a `card_labels` join table.

**Prompt 2 — Confirm and scope:**

```
Approved. Implement schema + migration first. Stop after
pnpm db:generate succeeds.
```

→ Two files changed, migration generated, tests pass.

**Prompt 3 — Contract:**

```
Now add tRPC procedures: labels.list, labels.create, labels.update,
labels.delete, cards.attachLabel, cards.detachLabel. Update
docs/features/cards.md.
```

**Prompt 4 — UI:**

```
Read the LabelChip pattern in packages/ui/src/badge.tsx if it exists,
otherwise design one. Add label management to the card detail
modal — apps/web/src/components/card-detail.tsx. Don't touch
anything else in that file.
```

**Prompt 5 — Test:**

```
Write a Playwright test: create label, attach to card, refresh,
label still shown.
```

**Prompt 6 — Commit:**

```
Run pnpm typecheck, lint, test. If green, draft a commit message
in conventional commits style.
```

**Total session:** ~90 minutes, ~80K tokens, one focused PR. Claude never sees the timer code, the reports code, or the realtime code — none of which are relevant.

---

## Context Budget Reference

A rule of thumb per session (200K window):

| Bucket                              | Budget   | Notes                 |
| ----------------------------------- | -------- | --------------------- |
| System + auto-loaded CLAUDE.md/docs | ~20K     | Keep CLAUDE.md tight  |
| Files Claude reads                  | ~40K     | 5-15 files            |
| Plan + back-and-forth               | ~20K     | Plan mode is cheap    |
| Generated code + edits              | ~50K     | The actual work       |
| Tool outputs (greps, test runs)     | ~30K     | Wastes the most       |
| **Headroom buffer**                 | **~40K** | Don't run to the wall |

When tool outputs balloon (long test failures, big greps), delegate to a subagent — its 50K of grep output collapses into a 500-token summary in your main context.

Even with a 1M context window, you should still work this way. Focused context produces better code, faster, cheaper. The 5-minute prompt-cache TTL means scrolling context still has cost; quality also degrades with noise.

---

## Common Pitfalls (and the fix)

| Pitfall                                     | Fix                                                                             |
| ------------------------------------------- | ------------------------------------------------------------------------------- |
| "Just one more thing" sessions              | Hard stop at one feature. Commit. `/clear`.                                     |
| CLAUDE.md becomes a 500-line wiki           | Cap at ~150 lines; push details into `docs/features/*` and link                 |
| Claude regenerates files it shouldn't touch | Be explicit: "Only edit X. List others and stop." Use plan mode.                |
| Tests are flaky / slow → Claude skips them  | Fix the tests, not the rule. A green suite is your only objective signal.       |
| Two features merged together in one PR      | One feature per branch. Even if it feels slower, debugging is 10× faster.       |
| Docs go stale                               | The audit ritual (Phase 5) every 5-10 sessions                                  |
| "It worked when Claude said done"           | Always run the feature in the browser. Type-check is necessary, not sufficient. |

---

## TL;DR Workflow

1. **Phase 0 once:** scaffold + CLAUDE.md + ARCHITECTURE.md + CI.
2. **Per feature:**
   1. Contract session (schema + tRPC signatures + feature doc) — small, careful
   2. Backend session (implement + tests)
   3. Frontend session (consume typed client)
   4. Polish session if needed
3. **Per session:** fresh `/clear`, point at 3-8 specific files, plan mode for non-trivial work, commit at the end.
4. **Every ~5-10 sessions:** audit ritual to catch drift.
5. **Always:** the type system + CI + Playwright are the safety net. Invest in them early; they pay back tenfold.

The discipline is: **the codebase is the truth, the docs are the index, and every session loads only the slice it needs.**
