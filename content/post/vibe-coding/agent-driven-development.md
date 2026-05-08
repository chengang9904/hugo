+++
date = '2026-05-08T04:12:12-07:00'
draft = false
title = 'Agent Driven Development'
categories = ['vibe-coding']
description = 'A practical tutorial for building large-scale web projects with code agents (Claude Code and similar).'
tags = ['agents', 'claude-code', 'development', 'best-practices']
+++

A complete guide to building large-scale web projects with code agents (Claude Code and similar). Read top to bottom; each section builds on the last.

---

## Introduction

### What this is

A practical handbook for engineering teams using code agents seriously — not as autocomplete, but as the primary interface for writing and reviewing code. It assumes you have an agent and a non-trivial codebase. It covers what to build _around_ the agent so the agent produces consistently good work.

### Who it's for

- Tech leads setting up agent workflows for a team
- Senior engineers on agent-driven projects who want fewer surprises
- Anyone running a long-running refactor or migration with an agent

### The thesis

> Most agent-driven productivity comes from infrastructure, not prompting. Better prompts produce slightly better output. Better infrastructure — clear specs, layered conventions, hooks, ADRs, validation scripts — produces dramatically better output, consistently, across every session.

A team that invests one week in infrastructure typically produces 2-3x more shippable code per agent-hour over the next quarter than a team with the same agent and no infrastructure. The infrastructure compounds. The prompting doesn't.

---

## Part 1: The Mental Model

### The development loop

Every agent-driven feature follows this loop:

```
intent → spec → plan → implementation → verification → review → merge
       ↑                                                            │
       └────────────────── learnings (memory, conventions) ─────────┘
```

Each arrow is a place errors compound. The mistake most teams make is collapsing `spec → plan → implementation` into a single prompt — the agent designs while it codes, and you can't tell where it went wrong. Keep these as separate artifacts in separate turns.

For trivial work (rename, bug fix, docs), skip to implementation. For anything touching > 3 files or > 1 module, run the full loop.

### The three meta-rules

These compound across every section that follows:

**1. Optimize for legibility, not cleverness.** Every choice — naming, structure, abstraction level — should privilege "easy for the next reader to understand" over "elegant by some other metric." Agents are readers too, and clear code makes them better.

**2. Make the right thing easy and the wrong thing hard.** If a convention requires constant vigilance, it will fail. Encode it in linters, hooks, types, or directory structure. The agent should fall into the right pattern by default.

**3. Treat your agent setup as a first-class part of the codebase.** `CLAUDE.md`, `specs/`, `.claude/`, `docs/adr/` — these are not "documentation," they are _infrastructure_. They get reviewed, refactored, and improved like any other code. Bad infrastructure produces bad output, even with great agents.

### The fix-upstream principle

If something feels broken, the fix is almost always upstream of where the symptom appears:

- Bad code? Look at the prompt.
- Bad prompt? Look at the plan.
- Bad plan? Look at the spec.
- Bad spec? Look at the conventions and `CLAUDE.md`.
- Wrong conventions? Look at the project's actual goals.

---

## Part 2: Setting Up the System

### CLAUDE.md hierarchy

Claude Code auto-loads `CLAUDE.md` from the working directory and every parent up to the repo root. Use this to put context where it's needed, not all at the root.

#### Root `CLAUDE.md` — the always-loaded file

Keep it under ~150 lines. Answer "what is this and what are the global rules":

```markdown
# project-name

Stack: Next.js 14 (App Router) + TypeScript + Tailwind.

## Commands

- `pnpm dev` — local dev (port 3000)
- `pnpm test` — Vitest
- `pnpm typecheck` — `tsc --noEmit`
- `pnpm lint` — ESLint, must pass before commit

## Conventions

- Server Components by default; mark client only when needed
- API routes in `app/api/**/route.ts`
- Shared types in `types/`, never duplicate
- Tailwind only — no CSS modules
- No default exports except Next.js page/layout/route files

## Hard rules

- Never edit `app/legacy/**`
- Never run migrations without `--dry-run` first
- Never commit without `pnpm typecheck && pnpm lint`

## Where to look

- Architecture decisions: `docs/adr/`
- Active specs: `specs/active/`
- Module-specific rules: each module has its own CLAUDE.md
```

What does **not** belong in root `CLAUDE.md`:

- File listings or directory trees (the agent can list)
- Code style minutiae (the linter enforces this)
- Git history (use `git log`)
- Module-specific rules (push them down to subdirectory `CLAUDE.md`)

#### Subdirectory `CLAUDE.md` — loaded only when working there

```
app/api/CLAUDE.md          # API route conventions, error shape, auth
components/CLAUDE.md       # component patterns, prop typing rules
lib/db/CLAUDE.md           # query patterns, transaction rules
specs/CLAUDE.md            # how specs work, where the template is
```

A line like "all queries in this directory must go through `withTransaction()`" in `lib/db/CLAUDE.md` prevents an entire class of agent mistakes — without bloating the root file every other agent invocation reads.

### The `specs/` system

A spec is your one chance to disambiguate before the agent makes irreversible-feeling choices. Without a spec, the agent will confidently produce 800 lines of code that solve the wrong problem.

#### Directory structure

```
specs/
  README.md                       # legend, lifecycle, ownership
  CLAUDE.md                       # tells the agent how specs work here
  _template/
    spec.md                       # canonical spec template
    plan.md                       # canonical plan template
    notes.md                      # canonical notes template
  conventions/                    # cross-cutting rules
    coding-style.md
    api-contracts.md
    testing.md
    accessibility.md
    security.md
  draft/                          # being written, NOT for implementation
  active/                         # approved, currently being built
    2026-05-08-card-detail-redesign/
      spec.md
      plan.md
      notes.md
      assets/
  blocked/                        # approved but waiting on something
  done/                           # completed, kept for reference
  archive/                        # rejected or deferred
```

Why this shape:

- **Lifecycle folders** are a cheap state machine. The agent can tell from the path whether a spec is authoritative.
- **Folder-per-active-spec** keeps plans, notes, and assets scoped.
- **Date-prefixed slugs** sort chronologically and read well in `git log`.
- **`_template/`** ensures specs don't drift in shape.
- **`conventions/`** is referenced by specs, not copied into them.
- **`archive/` keeps rejections** — when the same idea comes up again, the reasoning is one grep away.

#### The spec template

```markdown
---
title: <Short Title>
slug: <YYYY-MM-DD>-<kebab-slug>
status: draft | active | done | archived
owner: <person>
created: <YYYY-MM-DD>
target: <YYYY-MM-DD>
related:
  - specs/done/...
conventions:
  - specs/conventions/...
---

## Problem

One paragraph. What is broken or missing today, and for whom. Include
metrics or quotes if you have them. No solutions yet.

## Goals

Bulleted, measurable. Each goal should be verifiable after the fact.

## Non-Goals

**The most important section.** List things a reasonable reader might
assume are in scope, and rule them out.

## Approach

2–5 paragraphs. Shape of the solution, not the implementation. Name the
modules that will change. Call out the risky parts.

## Acceptance Criteria

Numbered, testable. These become the test plan.

1. <criterion>
2. <criterion>

## Open Questions

Things you don't know yet. Better to ship a spec with open questions
than to bake in the wrong assumption.

## Rollout

How this ships. Flag? Staged? What's the kill switch?

## Out of scope but worth noting

Adjacent ideas that came up. Not committed; recorded so they don't
get lost.
```

#### Spec quality bar

A good spec lets you predict what code the agent will produce before it produces it. That's the bar.

Rules of thumb:

- Short. A two-page spec gets read; a ten-page spec gets skimmed.
- Concrete acceptance criteria. "User can archive a card" is bad. "Clicking archive sets `archived_at`, removes the card from the list within 200ms, and shows an undo toast for 5s" is good.
- Honest non-goals. If you're tempted to write "we may also redesign X," either commit or rule it out. Maybe-scope is poison.
- Linked, not embedded, conventions. Reference `specs/conventions/testing.md`; don't copy it.

### Plans vs specs

The **spec** answers "what and why." The **plan** answers "how, in what order." A plan is generated _after_ the spec is approved.

Plan template:

```markdown
---
spec: spec.md
created: <YYYY-MM-DD>
---

## Files to change

- <path> — <change summary>

## Order (each step independently reviewable)

1. <step>
2. <step>

## Risks

- <risk and mitigation>

## What this plan does NOT do

- <out of scope from spec>
```

The plan is what you hand the agent for implementation. It's specific enough that you can sanity-check the diff against it.

### Architecture Decision Records (ADRs)

Specs say _what_ to build. ADRs say _why_ we build that way.

```
docs/adr/
  README.md
  _template.md
  0001-use-app-router.md
  0002-zustand-for-client-state.md
  0003-react-query-for-server-state.md
  0004-server-actions-over-trpc.md
  0005-tailwind-only.md
```

ADR template:

```markdown
---
id: <id>
title: <title>
status: accepted | superseded | deprecated
date: <YYYY-MM-DD>
supersedes: <id, if applicable>
---

## Context

What forces are at play. What we considered. Why this needs deciding now.

## Decision

What we chose. One paragraph, declarative.

## Consequences

Positive, negative, neutral. The "negative" section is the most
important — that's what the next person needs to know.

## Alternatives considered

- <option>: why rejected
```

When to write an ADR:

- Any choice that took > 1 hour of discussion
- Any choice that locks out alternatives
- Any choice that contradicts an industry default
- Any choice future agents will second-guess

When the agent proposes "let's use Recoil for this state," you don't argue from memory — you point to ADR-0002 ("Zustand for client state") and say "doesn't fit our convention; if you think we should change, write a superseding ADR first."

### `.claude/` configuration

#### Permissions (`.claude/settings.json`)

```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm test:*)",
      "Bash(pnpm typecheck)",
      "Bash(pnpm lint)",
      "Bash(pnpm build)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(gh pr view:*)",
      "Bash(gh pr list:*)"
    ],
    "ask": ["Bash(git push:*)", "Bash(gh pr create:*)"],
    "deny": ["Bash(git push --force*)", "Bash(rm -rf*)", "Bash(pnpm publish*)"]
  }
}
```

Read-only and idempotent commands shouldn't prompt. The `fewer-permission-prompts` skill scans your transcripts and suggests these — run it monthly.

#### Hooks

Hooks run on harness events and are the only way to _enforce_ automated behavior — not memory, not preferences.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": { "tool": "Edit", "pathGlob": "**/*.{ts,tsx}" },
        "command": "pnpm typecheck --pretty 2>&1 | tail -50",
        "outputBehavior": "feed-to-claude-on-failure"
      },
      {
        "matcher": { "tool": "Write", "pathGlob": "specs/active/**/spec.md" },
        "command": "node scripts/validate-spec.js \"$CLAUDE_FILE_PATH\"",
        "outputBehavior": "block-on-failure"
      }
    ],
    "PreToolUse": [
      {
        "matcher": { "tool": "Bash", "commandPattern": "git push" },
        "command": "scripts/check-not-main.sh",
        "outputBehavior": "block-on-failure"
      }
    ]
  }
}
```

Use the `update-config` skill to wire these — it knows the schema details.

#### Slash commands (`.claude/commands/`)

Slash commands are workflow accelerators. Keep them in `.claude/commands/`.

`spec.md`:

```markdown
---
description: Create a new spec from the template
---

Create a new spec under `specs/draft/` using `specs/_template/spec.md`.

1. Ask the user for the title (slug = today's date + kebab-case title)
2. Create `specs/draft/<slug>.md` with the template, fill frontmatter
3. Open the file at the "Problem" section
4. Don't fill in the body — that's the user's job
```

`promote-spec.md`:

```markdown
---
description: Promote a draft spec to active
---

1. Verify the spec has all required sections filled. If any missing,
   refuse and tell the user which.
2. Create `specs/active/<slug>/` directory
3. Move `specs/draft/<slug>.md` → `specs/active/<slug>/spec.md`
4. Update frontmatter: status: active
5. Generate `plan.md` and empty `notes.md`
6. Report paths created
```

`ship-spec.md`:

```markdown
---
description: Move a spec from active to done
---

1. Verify all acceptance criteria boxes are checked
2. Verify notes.md has at least one entry
3. Move `specs/active/<slug>/` → `specs/done/<slug>/`
4. Update frontmatter: status: done, shipped: <today>
5. Update refactor ledger if applicable
```

---

## Part 3: The Daily Workflow

A worked example, end to end. Imagine you're adding a "duplicate card" feature.

### Step 1 — From idea to spec

15-minute draft in `specs/draft/duplicate-card.md`:

```markdown
---
title: Duplicate Card Action
slug: 2026-05-08-duplicate-card
status: draft
owner: hassan
created: 2026-05-08
---

## Problem

Users frequently rebuild similar cards from scratch. Support data shows
~12% of new cards are near-copies of existing ones.

## Goals

- One-click duplicate from the card detail page
- Duplicated card opens immediately in edit mode with focus on title
- Title prefilled with "Copy of {original}"

## Non-Goals

- NOT bulk duplicate (separate spec if needed)
- NOT a "template" feature — duplicates are independent
- NOT duplicating attached files — only metadata
- NOT duplicating comments or activity history

## Approach

Add `POST /api/cards/:id/duplicate` that copies the row, prefixes the
title, returns the new id. Add a Duplicate button on `CardActions` that
calls it and navigates to `/cards/:newId?edit=1`.

## Acceptance Criteria

1. POST returns 201 with `{ id, title }` of the new card
2. New card has all metadata except `id`, `created_at`, `updated_at`
3. Title is `"Copy of " + original.title`, truncated to schema limit
4. Clicking Duplicate navigates to new card in edit mode
5. Title input is focused and selected on edit-mode load
6. Unauthorized user gets 403, missing card gets 404
7. New E2E test covers the happy path

## Open Questions

- [ ] Rate limit? Probably yes but maybe later — defer if not trivial

## Rollout

Behind `card_duplicate` flag. Internal at 100% on merge, public 50%
next day, full the day after if no errors. Kill switch: flip flag off.
```

Promote: move to `specs/active/2026-05-08-duplicate-card/spec.md`.

### Step 2 — From spec to plan

`specs/active/2026-05-08-duplicate-card/plan.md`:

```markdown
---
spec: spec.md
created: 2026-05-08
---

## Files to change

- `app/api/cards/[id]/duplicate/route.ts` — new POST handler
- `lib/cards/queries.ts` — add `duplicateCard(id, userId)`
- `lib/cards/queries.test.ts` — unit tests
- `components/cards/CardActions.tsx` — add Duplicate button
- `components/cards/CardActions.test.tsx` — test button click
- `tests/e2e/duplicate-card.spec.ts` — new E2E
- `lib/featureFlags.ts` — register `card_duplicate` flag

## Order (each step is independently reviewable)

1. `duplicateCard()` query + unit tests. PR-1.
2. Route handler + integration test. PR-2.
3. Flag registration. PR-3 (tiny).
4. UI button + component test, gated on flag. PR-4.
5. E2E test, gated on flag. PR-5.
6. Remove flag check after rollout. PR-6 (cleanup, separate).

## Risks

- Title truncation: schema is varchar(255), "Copy of " is 8 chars,
  so truncate original to 247.
- File attachments: must NOT copy. Verify in test that
  `card_attachments` has no rows for new card.

## What this plan does NOT do

- Bulk duplicate
- Template/link relationship
- File/comment duplication
- Rate limiting (deferred)
```

### Step 3 — Implementation

The prompt should reference the artifacts, not re-explain them:

```
Implement step 1 of the plan in
specs/active/2026-05-08-duplicate-card/plan.md.

Scope: only the duplicateCard() query and its unit tests. Do not touch
the route handler, UI, or flag registration — those are later steps.

Read the spec for context, especially the acceptance criteria around
title prefix, truncation, and which fields to copy.

Write the unit test first, then the implementation. Tests must cover:
- Happy path: all metadata copied, attachments not copied
- Title truncation when original is long
- Authorization: user can only duplicate their own cards
- Original card not found returns null

Use the existing query patterns in lib/cards/queries.ts. Don't introduce
new abstractions.

Report:
1. The new function signature
2. The list of test cases written
3. Any deviations from the plan and why
```

### Step 4 — Verification

The agent's end-of-turn summary is **intent**, not result. Verify.

Cheap verifications (do every time):

```bash
# What did the agent actually change?
git diff --stat

# Any new 'as any', '@ts-ignore', or '.skip'?
git diff | grep -E "as any|@ts-ignore|\.skip"

# Did test count go up?
pnpm test --reporter=json | jq '.numTotalTests'

# Run the new tests
pnpm test lib/cards/queries.test.ts

# Type check
pnpm typecheck
```

Worth-it verifications for non-trivial changes:

- **UI changes:** start the dev server, click through the feature. Type-check passing ≠ feature working — there is no automated substitute for this.
- **API changes:** hit the endpoint with `curl`. Confirm response shape.
- **Data changes:** run migration on a copy of prod data, not just dev fixtures.

### Step 5 — Review and ship

Review checklist:

```
[ ] Does the diff implement every acceptance criterion?
[ ] Are there changes outside the planned files? Justified?
[ ] Any new `as any`, `@ts-ignore`, `.skip`, `.only`?
[ ] Test count: did it go up by at least the number of new behaviors?
[ ] Did any existing tests change? Read each change.
[ ] Comments: any restate the code? Delete.
[ ] Error handling: any catch-and-default that hides real errors?
[ ] Does the new code match patterns from existing files in the same dir?
[ ] Run the feature in a browser / hit the endpoint.
```

Use `/review` for fast self-review before pushing. Reserve `/ultrareview` for high-stakes PRs (auth, data, shared infra).

After merge:

- Update `notes.md` with anything unexpected during implementation
- Move spec to `done/`
- Update memory if anything was non-obvious
- Update refactor ledger if applicable

### The `notes.md` artifact

The most undervalued artifact in this whole system. Where future-you (and future agents) learn from this implementation. Three months later when someone duplicates this pattern, they read this and skip the same mistake.

```markdown
## 2026-05-08 — step 1

- Agent introduced `truncateTitle()` helper instead of inlining.
  Reasonable, kept it.
- Agent's first attempt copied `created_at`. Spec was clear this should
  be regenerated. Caught in review, fixed.
- Authorization: implemented as `where userId = ctx.userId` in the
  query rather than a separate auth check. Matches pattern in
  `getCard()`.
```

---

## Part 4: Working Effectively with Agents

### Prompt engineering basics

#### The implementation prompt shape

```
Goal: <one sentence>
Read: <files/docs the agent needs>
Scope: <what's in>
Out of scope: <what's tempting but excluded>
Constraints: <hard rules>
Verify: <what the agent should run before reporting>
Report: <what the agent should tell you>
```

The "Report" section matters more than people think. Without it, the agent gives you marketing copy ("successfully implemented the feature"). With it, you get information you can act on.

#### The investigation prompt shape

```
Question: <one sentence>
Why I'm asking: <so the agent can judge edge cases>
Already ruled out: <so it doesn't repeat work>
Depth: quick | medium | thorough
Output: <length cap, format>
```

"Why I'm asking" is the single most valuable line. It turns a narrow lookup into an answer that addresses the actual question.

#### Anti-patterns in prompts

- **"Implement the feature."** Too vague. Agent invents scope.
- **"Make it production ready."** Means nothing measurable.
- **"Be careful."** Agents can't be more careful on demand. Be specific about what to verify.
- **"Use best practices."** Whose? Specify the rule, not the vibe.
- **"Make it match the existing style."** Better: "match the patterns in `lib/cards/queries.ts`."

### Context management

The 1M context window is a budget, not free. Once you hit ~60% utilization, agent quality drops noticeably — it forgets earlier instructions, repeats searches, gets confused about file state.

#### Detecting bloat

- Agent re-reads files it already read this session
- Agent asks for information you already gave it
- Responses get more generic and hedged
- You find yourself repeating constraints

#### Strategies

- **Push context to files.** A 500-line `CLAUDE.md` costs ~2K tokens once. Repeating those 500 lines across 20 prompts costs 40K tokens.
- **Fork for research.** Tool output stays in the fork, not your main thread.
- **`/compact` strategically.** Before starting a new task in the same session.
- **`/clear` between tasks.** If task A is done and task B is unrelated, just clear.
- **Avoid pasting large files.** Reference them by path.
- **Avoid screenshot-heavy sessions for code work.** Images are expensive.

### Communication patterns

#### Calibrated uncertainty

Agents will confidently propose answers. You should respond with calibrated confidence too:

- **"I think X but I'm not sure — verify before acting."**
- **"This is wrong because Y, but I might be wrong about Y."**
- **"I want X, but if you see a reason it's bad, push back."**
- **"Don't change anything yet — explain what you'd do."**

#### The "explain it back" pattern

Before letting the agent implement something non-trivial:

> "Before you start, summarize what you understand the task to be. List the files you expect to change and any acceptance criteria you think apply. Don't write any code."

If the summary matches your intent, proceed. If it doesn't, the spec/prompt was unclear — fix it before any code is written. You catch 80% of misunderstandings here, at zero cost.

#### Productive disagreement

When the agent pushes back, take it seriously. Sometimes it's right. Useful prompts:

- **"You said X. I think Y. Here's why: [reason]. What am I missing, or do you want to change your view?"**
- **"Walk me through your reasoning step by step."**
- **"What evidence would change your mind?"**

When the agent says "you're right, I'll do Y" — be skeptical. Sometimes that's the right answer; sometimes it's the agent capitulating because you sound certain. Push: "Are you actually convinced, or just deferring?"

#### When to interrupt

Stop the agent immediately if:

- It's about to run a destructive command you didn't authorize
- It's modifying files outside the planned scope
- It's three iterations into a fix-test-fail loop with no progress
- Its summary diverges from what the diff actually shows

Don't wait politely for it to finish a wrong direction.

### Multi-agent coordination

#### Collision modes

1. Same branch, different agents → merge conflicts
2. Same database, different agents → fixture poisoning
3. Same context, different agents → rate limits
4. Same spec, different agents → diverging implementations

#### Coordination protocol

**One spec, one branch, one agent at a time.** Worktrees enforce this:

```bash
git worktree add ../proj-duplicate-card feat/2026-05-08-duplicate-card
```

**Database isolation per agent:**

```typescript
const schemaName = `test_${process.env.JEST_WORKER_ID || "main"}_${Date.now()}`;
process.env.DATABASE_URL = `${BASE_URL}?schema=${schemaName}`;
```

**Parallel forks for research, not implementation.** Forks are great for "audit X, audit Y, audit Z" run in parallel. They are bad for "implement X, implement Y, implement Z" because the diffs need to interleave.

#### When parallel makes sense

```
Single message, three forks:
  Fork A: "audit specs/active/, report which are ready to merge"
  Fork B: "audit tests/, report any committed .skip or .only"
  Fork C: "audit dependencies, report any with security advisories"

Coordinator (you) reads three notifications, synthesizes, decides.
```

When in doubt: serial is safe; parallel is for I/O-bound research.

---

## Part 5: Code Structure That Helps Agents

This is the most underrated lever. Agents perform dramatically better in well-structured codebases.

### Co-located tests

```
components/cards/
  CardActions.tsx
  CardActions.test.tsx
  CardActions.stories.tsx
```

Agents find tests reliably when they live next to code. A separate `tests/` mirror requires the agent to track two trees and they often miss tests when refactoring.

### Explicit module boundaries

Each top-level directory should have an `index.ts` that re-exports its public surface. This signals to the agent (and humans) what's API and what's internal.

```typescript
// lib/cards/index.ts
export { getCard, duplicateCard, listCards } from "./queries";
export type { Card, CardInput } from "./types";
// Note: `lib/cards/internal/*` is NOT re-exported — internal use only
```

### Shallow trees over deep ones

`components/cards/CardDetail.tsx` is easier for an agent to navigate than `src/features/cards/components/detail/CardDetail/CardDetail.tsx`. Each extra level is friction.

### Predictable file shapes

If every API route looks like this:

```typescript
export const POST = withAuth(async (req, { params, user }) => {
  const { id } = await params;
  const result = await duplicateCard(id, user.id);
  if (!result) return notFound();
  return NextResponse.json(result, { status: 201 });
});
```

…the agent generates new ones correctly the first time. Inconsistent route shapes mean the agent has to guess.

### Type-first interfaces

Define types before implementation. The agent can write code against types it can see; it can't write code against types you mean.

```typescript
// lib/cards/types.ts
export type Card = {
  /* ... */
};
export type CardInput = Omit<Card, "id" | "createdAt" | "updatedAt">;
export type DuplicateCardResult = { id: string; title: string } | null;
```

### Avoid magic

Decorators, dynamic imports, runtime metaprogramming, monkey-patching — all cost agent reasoning quality. Boring, explicit code is agent-friendly code. (It's also human-friendly. The two correlate.)

### Index files for large modules

For modules with many files, add an `INDEX.md`:

```markdown
# lib/cards/INDEX.md

## Files

- queries.ts — DB queries: getCard, listCards, duplicateCard, archiveCard
- mutations.ts — write paths with side effects
- types.ts — Card, CardInput, CardEvent
- permissions.ts — canRead/canWrite/canShare for cards

## Patterns

All queries take userId as second arg, return null if not authorized.
Mutations emit `card:*` events via the event bus.

## Where to look first

- New query? queries.ts, follow getCard pattern
- New mutation? mutations.ts, follow archiveCard pattern
```

A 30-second investment that pays off every session.

---

## Part 6: Conventions Worth Standardizing

### Performance and accessibility budgets

`specs/conventions/performance.md`:

```markdown
## Performance budgets

Build fails if any of these exceed:

- Initial JS bundle: 180KB gzipped
- Per-route JS: 60KB gzipped
- LCP on /cards/[id]: 2.0s p75
- TTI on /cards: 2.5s p75
- API p95: 300ms (excluding cold starts)
```

`specs/conventions/accessibility.md`:

```markdown
## A11y budgets

- Lighthouse a11y score: minimum 95 on every page
- axe-core: zero violations of severity "serious" or "critical"
- Keyboard: every interactive element reachable via keyboard

Common agent mistakes (will fail review):

- `<div onClick>` instead of `<button>`
- Missing `alt` on images
- Color contrast below 4.5:1
- Form inputs without associated `<label>`
- Modals that don't trap focus
```

A pre-commit hook running `axe-core` on changed routes catches most of these.

### Security and authentication

```markdown
## Auth conventions

All authenticated routes go through `requireUser()`.

`requireUser()` is the ONLY place that reads session cookies.

Authorization is per-resource, checked in the data layer:
❌ const card = await db.cards.findById(id)
if (card.userId !== user.id) throw new Forbidden()
✅ const card = await db.cards.findFirst({
where: { id, userId: user.id }
})
```

#### Security checklist for auth-touching PRs

```
[ ] No new endpoint mutating data via GET
[ ] No `eval`, `new Function`, dynamic `require()`
[ ] No user input reaches `dangerouslySetInnerHTML` without sanitization
[ ] No user input reaches a shell, file path, or SQL string
[ ] All redirects validated against allowlist
[ ] Cookies have `Secure`, `HttpOnly`, `SameSite` set
[ ] Auth checks at data layer, not just route layer
[ ] No secrets in code, env files, or test fixtures
[ ] Rate limiting on auth endpoints
[ ] Password reset tokens single-use and time-limited
```

Run `/security-review` on auth-touching PRs.

### Database and migrations

The highest-blast-radius area. Hard rules in `lib/db/CLAUDE.md`:

```markdown
NEVER:

- Run a migration without `--dry-run` first
- Use raw SQL for user-facing queries
- Disable RLS, FK constraints, or NOT NULL "temporarily"
- Drop columns/tables in the same migration that adds the replacement

ALWAYS:

- Migrations are forward-only. To revert, add a new migration.
- Schema changes follow expand → migrate → contract:
  1. Add new column/table (nullable/default), deploy
  2. Backfill data, verify
  3. Switch reads/writes, deploy
  4. Remove old column/table in a later release
- Indexes added with `CREATE INDEX CONCURRENTLY` on tables > 10k rows
```

Five PRs for "rename a column" is not overkill. It's the difference between "deploy on a Wednesday" and "production outage at 3am."

### State management

```markdown
## State management

Server state (data from the API):

- React Query for client-side reads
- Server Components for server-side reads
- Never store server state in Zustand/useState — it goes stale

Client state (UI-only, ephemeral):

- useState for component-local
- Zustand for cross-component
- URL state for anything user might bookmark/share

Form state:

- react-hook-form + Zod
- Never useState for forms with > 2 fields

URL is state too:

- Filters, sort, pagination, tab selection → URL params
```

The "URL is state too" rule is the most undervalued. Agents will reach for `useState` to track which tab is selected, breaking deep links and back button.

### Forms

```markdown
Stack: react-hook-form + Zod + Server Actions.

Pattern (every form follows):

1. Schema in `<Form>.schema.ts`, co-located
2. Form uses `useForm({ resolver: zodResolver(schema) })`
3. Submit calls a Server Action that re-validates with the SAME schema
4. Display errors:
   - Field-level: from react-hook-form
   - Form-level: from Server Action's `{ ok: false, error }` response
   - Never `alert()` for user-visible errors
5. Loading state via `useFormStatus()` inside `<SubmitButton>`

NOT allowed:

- Validation only on the client
- Different schemas client vs server
```

### Observability

```markdown
## Observability

Three pillars:

### Logging

- Logger: `logger` from `lib/logger.ts` (pino)
- Levels: `debug` | `info` | `warn` | `error`
- Context: every log includes `userId` and `requestId` if available
- PII: never log `email`, `name`, raw request bodies

### Errors

- Sentry for capture
- Every Server Action wraps with `withErrorReporting`
- Never swallow errors silently. If you catch, you log.

### Metrics

- StatsD/OpenTelemetry counter for: each Server Action, each Route Handler
- Naming: `<domain>.<verb>.<status>` e.g., `cards.duplicate.success`
- Cardinality: never include user ID or card ID as a tag

For every new user-facing action:

1. Log entry at start
2. Counter on success
3. Counter on each known error path
4. Histogram of duration
```

---

## Part 7: The Refactor Playbook

A "refactor" project is mostly about _not breaking things while changing them_. Agents can be unusually good or unusually bad depending on setup.

### Strangler fig, not big bang

Keep old code working while new code is added next to it. Agents handle "add new code" much better than "edit old code in place." The Next.js Pages → App Router migration naturally fits this.

### Make the new path opt-in via flag

Not just a feature flag for users — a routing flag for _you_. New page handles `?v=2`, old handles default. Compare side-by-side. Once new is ready, swap defaults. Agents thrive when they can build the new without breaking the old.

### Tests as the contract

Before refactoring a module, write characterization tests against its current behavior. Then the agent can refactor freely — if tests pass, behavior is preserved. Without tests, you're trusting the agent to preserve invariants it can't see.

### Type-driven extraction

Extract types first, in their own PR. Once `Card`, `CardInput`, `CardEvent` are well-typed, refactoring code that uses them is much safer because the type system catches drift.

### One pattern at a time

"Migrate all data fetching to Server Components" — fine.

"Migrate data fetching AND restructure components AND swap the styling system" — chaos.

Agents need a single transformation rule per pass.

### Codemod for mechanical, agent for judgment

If the refactor is mechanical (rename, move, change import), use jscodeshift or ts-morph — deterministic and reviewable as a single diff. Reserve the agent for the parts that need judgment.

### Boundary first for legacy code

Before touching legacy code, draw a boundary around it. `app/legacy/**` is its own region with its own `CLAUDE.md`:

```markdown
# Legacy code

This directory is being phased out.

- Do NOT add new features here. New features go in `app/`.
- Do NOT refactor for style. Edits should be minimal.
- DO fix bugs that affect users today.
- DO add deprecation comments when something is replaced.
```

### Refactor ledger

Keep `docs/refactor-ledger.md` as a running record:

```markdown
# Pages → App Router Migration

## Done

- [x] /login (2026-04-02)
- [x] /signup (2026-04-04)
- [x] /cards (list view) (2026-04-15)

## In progress

- [ ] /cards/[id] (detail) — spec: 2026-05-08-card-detail-redesign

## Not started

- [ ] /settings/\*
- [ ] /admin/\* (deferred)

## Conventions established so far

- Server Component by default
- Data fetching via co-located server actions
- Loading states via `loading.tsx`
```

The agent reads this and knows what's done, what patterns are established, what to follow.

---

## Part 8: Anti-Patterns

The recurring failure modes:

1. **The mega-prompt.** "Build the whole feature" with no spec. You get 1500 lines of plausible-looking code that solves a different problem than you wanted. **Fix:** spec → plan → implement.

2. **Trusting the summary.** Agent says "all tests pass." Tests pass because the agent skipped the failing one. **Fix:** verify diff, not summary.

3. **Letting context bloat.** 200K tokens of tool output later, the agent has forgotten the original goal. **Fix:** fork research, use `/clear` aggressively, push context to `CLAUDE.md`.

4. **No non-goals.** Every spec gets implemented plus "while I'm here" cleanups. **Fix:** explicit non-goals, enforced at review.

5. **Permission prompt fatigue.** You start clicking "yes" without reading. **Fix:** allowlist read-only commands, reserve prompts for actions that matter.

6. **Memory as scratchpad.** Saving "currently working on X" — that's task state, goes stale instantly. **Fix:** memory is for _durable_ facts.

7. **One giant CLAUDE.md.** 800 lines at the root, loaded on every turn, ignored by humans. **Fix:** layer CLAUDE.md by directory.

8. **Specs as wishlist.** Specs that say "we should also consider…" without committing. **Fix:** cut everything that isn't acceptance criteria or non-goals.

9. **Skipping the human gate.** Agent produces spec, plan, and code in one chain. No one ever approved the spec. **Fix:** human approval between artifacts.

10. **Agent-generated commit messages with no review.** Often wrong about _why_. **Fix:** edit the message, or write it yourself.

### Failure recovery

Sometimes the agent produces broken code or makes changes you didn't want.

#### Triage first

Before reverting:

1. Is the diff _partially_ correct? Salvage the good parts.
2. Is the bug obvious? Just fix it forward.
3. Is the approach wrong? Revert and re-prompt.

Most teams over-revert. A 200-line diff with one bad function doesn't need to start over.

#### When to revert

- Agent fundamentally misunderstood the spec
- Diff touches files outside scope and you can't easily separate them
- Multiple changes interact in ways that are hard to debug

#### How to revert without losing learning

```bash
git diff > /tmp/agent-attempt-1.patch
git reset --hard HEAD
```

When re-prompting, _include what went wrong_: "Previous attempt did X, which violated constraint Y. Don't repeat that."

#### When the agent gets stuck in a loop

Agent makes a change, runs tests, sees a failure, makes another change, runs tests, sees a different failure…

Stop the loop. Take over manually for 5 minutes. The agent is missing context that would unlock progress. Once you understand the missing context, _write it down_ (in `CLAUDE.md` or `notes.md`) so future agent sessions don't re-discover it.

---

## Part 9: Maintaining the System

### Daily rhythm

**Morning (planning)**

- Read `git log` from yesterday
- Skim `specs/active/`
- Pick the next thing
- If no spec exists, write the draft (15-30 min)

**Midday (deep work)**

- One spec → one branch → one focused agent session
- Verify diff before committing
- Commit, push, open PR
- While CI runs: start next spec or review someone else's PR

**Late afternoon (cleanup)**

- Update `notes.md`
- Move shipped specs to `done/`
- Update memory if anything was non-obvious

### Friday hygiene

- Read your week's specs and PRs as a batch
- Look for patterns: what failed? what didn't?
- Update `CLAUDE.md` or ADRs based on what you learned
- Archive anything stale

### Quarterly pruning

- Memory: delete entries that haven't been useful
- `CLAUDE.md` files: anything obsolete?
- ADRs: any superseded but not marked?
- Specs in `archive/`: still informative? if not, delete
- Slash commands: any unused?
- Permissions: still right? run `fewer-permission-prompts` again

### Annual audit

- Read your last 12 months of `done/` specs as a batch. What patterns emerge?
- Read your last 12 months of postmortems. What recurring categories?
- Read your ADRs. Which need superseding?
- Read your `CLAUDE.md`s. Are they still the rules you actually follow?

### Continuous evolution

The system is not finished. It's a living thing. Every spec teaches the system. Every postmortem teaches the system. Every "I had to repeat that to the agent again" teaches the system.

Teams that make this work treat the agent infrastructure as a product they're building alongside their actual product. The ones that don't end up with a stale `CLAUDE.md` from project init and a confused agent six months later.

---

## Appendix A: Templates

### Spec template

See Part 2 / Spec Template above.

### Plan template

See Part 2 / Plans vs specs above.

### Bug spec template

```markdown
---
title: <short description>
slug: <date>-bugfix-<short>
status: active
owner: <you>
created: <date>
severity: P0 | P1 | P2 | P3
---

## Symptom

What the user sees / what's broken.

## Reproduction

1. Step
2. Step
3. Bug

## Root cause

Specific. NOT "the code is broken."

## Fix

What you'll change.

## Why this fix and not another

Document alternatives considered.

## Blast radius

- What else uses this code?
- What else might be affected by this fix?

## Tests

- Regression test that fails before fix, passes after

## Rollout

Usually "ship in next release," but P0 may require hotfix.
```

### ADR template

See Part 2 / Architecture Decision Records above.

### Postmortem template

```markdown
---
date: <YYYY-MM-DD>
duration: <minutes>
severity: SEV-1 | SEV-2 | SEV-3
authors: <people>
status: complete
---

## Summary

One paragraph.

## Timeline (UTC)

- <time> — first alert
- <time> — engineer ack
- ...

## Root cause

Specific. Not "deployment issue."

## Why our safeguards didn't catch it

- <gap>

## What went well

- <thing>

## What went badly

- <thing>

## Action items

Each becomes a spec or ADR:

- [ ] Spec: <link>
- [ ] ADR: <link>
- [ ] Runbook: <link>
```

---

## Appendix B: Scripts

### `scripts/validate-spec.js`

```javascript
#!/usr/bin/env node
const fs = require("fs");

const REQUIRED_FRONTMATTER = ["title", "slug", "status", "owner", "created"];
const REQUIRED_SECTIONS = [
  "## Problem",
  "## Goals",
  "## Non-Goals",
  "## Approach",
  "## Acceptance Criteria",
  "## Rollout",
];
const FORBIDDEN_PHRASES = ["TBD", "TODO", "fill this in", "lorem ipsum"];

const file = process.argv[2];
if (!file) {
  console.error("Usage: validate-spec.js <path-to-spec.md>");
  process.exit(2);
}

const content = fs.readFileSync(file, "utf8");
const errors = [];

const fmMatch = content.match(/^---\n([\s\S]*?)\n---/);
if (!fmMatch) {
  errors.push("Missing frontmatter");
} else {
  const fm = fmMatch[1];
  for (const key of REQUIRED_FRONTMATTER) {
    if (!new RegExp(`^${key}:`, "m").test(fm)) {
      errors.push(`Missing frontmatter field: ${key}`);
    }
  }
}

for (const section of REQUIRED_SECTIONS) {
  if (!content.includes(section)) {
    errors.push(`Missing section: ${section}`);
  }
}

for (const phrase of FORBIDDEN_PHRASES) {
  if (content.toLowerCase().includes(phrase.toLowerCase())) {
    errors.push(`Forbidden placeholder phrase: "${phrase}"`);
  }
}

if (errors.length > 0) {
  console.error(`Spec validation failed for ${file}:`);
  errors.forEach((e) => console.error(`  - ${e}`));
  process.exit(1);
}

console.log(`OK: ${file}`);
```

### `scripts/check-test-discipline.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

base=${1:-origin/main}

new_anys=$(git diff "$base"..HEAD -- '*.ts' '*.tsx' \
  | grep -c '^+.*as any' || true)
new_ignores=$(git diff "$base"..HEAD -- '*.ts' '*.tsx' \
  | grep -cE '^\+.*(@ts-ignore|@ts-expect-error)' || true)
new_skips=$(git diff "$base"..HEAD -- '*.test.ts' '*.test.tsx' '*.spec.ts' \
  | grep -cE '^\+.*(\.skip|\.only|xit|xdescribe)' || true)

fail=0
if [[ "$new_anys" -gt 0 ]]; then
  echo "FAIL: $new_anys new 'as any' introduced"
  fail=1
fi
if [[ "$new_ignores" -gt 0 ]]; then
  echo "FAIL: $new_ignores new @ts-ignore introduced"
  fail=1
fi
if [[ "$new_skips" -gt 0 ]]; then
  echo "FAIL: $new_skips new .skip / .only introduced"
  fail=1
fi

exit "$fail"
```

### `scripts/check-not-main.sh`

```bash
#!/usr/bin/env bash
set -euo pipefail

branch=$(git rev-parse --abbrev-ref HEAD)

if [[ "$branch" == "main" || "$branch" == "master" ]]; then
  echo "Refusing to push directly to $branch."
  exit 1
fi
```

---

## Appendix C: Definition of Ready / Done

### Definition of Ready

A spec is "ready" for implementation when ALL of the following are true:

- [ ] Frontmatter complete (status, owner, target date, related links)
- [ ] Problem stated in business terms (not "the code is bad")
- [ ] Goals are measurable
- [ ] Non-Goals explicitly list what's NOT in scope
- [ ] Approach names the modules/components that will change
- [ ] Acceptance Criteria are testable
- [ ] Open Questions are answered or explicitly deferred
- [ ] Rollout plan exists, including kill switch
- [ ] Plan generated and reviewed
- [ ] Spec linked from any related ADR or convention

Specs that don't meet this bar stay in `draft/`.

### Definition of Done

A spec is "done" when ALL of the following are true:

- [ ] All acceptance criteria pass in production
- [ ] Tests cover each acceptance criterion
- [ ] Type-check, lint, and tests pass on main
- [ ] No new `@ts-ignore`, `as any`, or `.skip` introduced
- [ ] Telemetry/logging in place to detect regressions
- [ ] Feature flag (if any) is at intended rollout state
- [ ] Spec moved to `specs/done/`
- [ ] `notes.md` captures any non-obvious decisions
- [ ] Memory updated if anything was non-obvious or surprising
- [ ] Refactor ledger updated (if applicable)
- [ ] Old code marked for deletion (with date) if this replaced something

---

## Appendix D: Onboarding Checklist

### Day 1 reading list

1. `README.md`
2. `CLAUDE.md`
3. `specs/README.md`
4. `specs/conventions/*` (skim)
5. `docs/adr/README.md` + 2-3 load-bearing ADRs
6. 3 specs in `specs/done/` chosen by your buddy

### Day 1 doing

- Pair on one trivial fix end-to-end with a buddy
- Open your first PR (anything — fix a typo)

### First-week milestones

- [ ] Ship one bug fix
- [ ] Ship one small feature with a spec
- [ ] Review one PR using `/review`
- [ ] Save one memory entry about yourself

### What new people get wrong

- **Skipping specs for "small" things.** Then "small" turns out to be a 6-file change. Default to writing a spec.
- **Trusting the agent's summary.** Cure: pair on the first 5 agent PRs.
- **Letting context bloat.** Cure: teach `/clear` and `/compact` early.
- **Memory hoarding.** Cure: show what NOT to save.

---

## Appendix E: Where to Start

If you want to do exactly one thing this week to improve your agent-driven workflow, in order of leverage:

1. Write a sharp `CLAUDE.md` at the root and one in each major module (1 day)
2. Build the `specs/` system with templates and one filled-in example (1 day)
3. Add `.claude/settings.json` permissions + 3-5 hooks (half day)
4. Write 3-5 ADRs for decisions you've already made implicitly (half day)
5. Add `validate-spec.js` and `check-test-discipline.sh` scripts (1 hour)
6. Write 3-5 reusable prompts in `prompts/` (1 hour)

Six items, one week, finished. Everything else is gardening on top of this.

---

## Closing thought

Agents are amplifiers. They amplify what you give them — clear specs become great code, vague specs become confident garbage. The infrastructure described in this tutorial is not overhead; it's the leverage point.

Build the system, then let the system do the work.
