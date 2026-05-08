---
title: "Git Workflow with Code Agents"
linkTitle: "Git Workflow"
date: 2026-05-08
lastmod: 2026-05-08
draft: false
description: "Practical Git workflow patterns for projects developed with AI code agents — branches, worktrees, dev branches, and merge vs. rebase."
summary: "How to structure branches, use worktrees for parallel agents, decide whether you need a dev branch, and choose between merge and rebase when integrating agent-driven work."
author: "Chen Gang"
tags:
  - git
  - workflow
  - code-agents
  - best-practices
categories:
  - tutorial
keywords:
  - git workflow
  - code agents
  - claude code
  - worktrees
  - rebase vs merge
weight: 10
toc: true
showToc: true
TocOpen: true
---

## Why this guide exists

Code agents change the economics of writing code: they produce diffs fast, sometimes faster than you can read them. That makes the *Git workflow around the agent* — how you isolate its work, review it, and integrate it — more important than the workflow you'd use working solo.

This tutorial walks through a practical setup: how to use branches, when worktrees help, whether you need a `dev` branch, and how to land agent-authored work on `main`.

## Core principles

Four rules that pay for themselves on every agent session:

1. **Protect `main`.** Agents never commit or push directly. All work lands via a PR you review. Configure branch protection: required PR review, required CI, no force-push.
2. **One task = one branch.** Each agent session gets its own short-lived branch. If the agent goes off the rails, you throw the branch away with no blast radius.
3. **Commit checkpoints frequently.** Small, atomic commits during development let you `git bisect`, revert, or cherry-pick when an agent makes a bad turn three steps in. Don't let the agent pile 2,000 LOC into one commit.
4. **Review the diff, not the agent's summary.** Agents reliably overstate what they did. `git diff main...HEAD` is ground truth.

## Branch layout

A simple, flat layout works for almost every project:

| Branch              | Purpose                                                                 |
| ------------------- | ----------------------------------------------------------------------- |
| `main`              | Always green, deployable. Protected.                                    |
| `feat/<short-desc>` | New feature work.                                                       |
| `fix/<bug>`         | Bug fixes.                                                              |
| `refactor/<area>`   | Refactors with no behavior change.                                      |
| `agent/<task>`      | Optional prefix for branches an agent originated, if you want it tagged. |

Naming with a prefix makes it obvious at a glance which branches are real work and which are agent-driven experiments.

### Throwaway / spike branches

When you let an agent explore an approach you may not keep, branch off `main`, expect to discard the branch, and don't bother polishing history. The whole point is that the branch is cheap.

## Do you need a `dev` branch?

**For most projects, no.** A `dev` branch is a holdover from Git Flow that adds overhead modern workflows rarely need.

### Skip it when

You do trunk-based development: short-lived feature branches → PR → `main`. Your safety comes from CI, PR review, and feature flags — not from a second integration branch. This is the default for SaaS, web apps, internal tooling, and most agent-driven workflows.

### Keep it when

A staging branch earns its keep in a few specific cases:

- **Versioned releases with a QA gate.** Tagged releases for desktop apps, libraries, mobile, or embedded — features integrate and get tested together before being cut.
- **Strict separation between "merged" and "deployed."** When `main` must reflect exactly what's live in production and you can't use tags or environment branches.
- **Many contributors with slow review cycles.** Though feature flags usually solve this better than an extra branch.

**Rule of thumb:** if you can't name a concrete thing `dev` would protect against that CI + PR review + feature flags don't already handle, you don't need it.

## Worktrees for parallel agents

When you run multiple agents simultaneously, a single working directory becomes a contention point: agents fight over the working tree, lockfiles, `node_modules`, and build output.

`git worktree` solves this — it lets you check out multiple branches into separate directories from the same repository:

```bash
# from your main repo directory
git worktree add ../card-flow-feat-auth feat/auth
git worktree add ../card-flow-fix-cache fix/cache-bug

# each agent works in its own directory, on its own branch
# shared .git, independent working trees
```

When the branch is merged or abandoned, remove the worktree:

```bash
git worktree remove ../card-flow-feat-auth
```

Claude Code's `Agent` tool supports `isolation: "worktree"` for exactly this reason — it spins up an isolated worktree per agent automatically.

## Merging worktree branches: merge or rebase?

Worktrees don't change the integration question — they're just a way to have multiple branches checked out at once. The merge/rebase decision is the same as for any feature branch. You have three options:

### 1. Squash-merge — recommended default for agent work

Collapse the whole feature branch into one commit on `main`. The granular checkpoint commits the agent produced were useful for *your* bisect/revert safety during development; they're noise in `main`'s history.

```bash
git checkout main
git merge --squash feat/auth
git commit -m "feat(auth): add OAuth login flow"
```

Or via GitHub: use **Squash and merge** as the default merge strategy.

### 2. Rebase + fast-forward

Replay each commit onto the tip of `main`, then fast-forward. Linear history with every commit preserved. Only worth it if the individual commits are themselves meaningful — which agent-authored commits usually aren't.

```bash
git checkout feat/auth
git rebase main
git checkout main
git merge --ff-only feat/auth
```

### 3. Merge commit (`--no-ff`)

Preserves the branch topology with an explicit merge commit. Useful when you want to see "this group of commits landed together," but creates a busier history. Most teams have moved away from this for feature work.

### What I'd actually do

1. **Rebase your branch onto `main` *before* merging** to resolve conflicts on your branch (not in a merge commit) and ensure CI runs against the version that will actually land.
2. **Then squash-merge** for features and bug fixes. One commit per PR on `main`, with a clean message.
3. **Exception:** if a PR genuinely contains multiple independent logical changes you want preserved, either split it into multiple PRs (preferred) or rebase-merge to keep the individual commits.

So the combo: **rebase onto `main` to update, squash-merge to land.**

## Don'ts

- **Don't rebase a branch someone else is working on** — including another agent in another worktree. Rebasing rewrites history; anyone with the old commits will have a bad time. Rebase is safe only on branches you own.
- **Don't merge `main` *into* your feature branch** to "catch up." It pollutes the branch with merge commits and makes the eventual diff harder to read. Rebase onto `main` instead.
- **Don't let agents bypass pre-commit hooks** with `--no-verify`. Hooks (lint, typecheck, tests) catch a large fraction of agent slop before it commits.
- **Don't trust the agent's PR summary over the diff.** Read the diff. Always.
- **Don't let agents force-push shared branches.** Force-push on a private feature branch is fine; on anything anyone else touches, it's not.

## Putting it together

A complete agent-driven feature flow:

```bash
# 1. create an isolated worktree for the agent
git worktree add ../proj-feat-search feat/search

# 2. agent works in ../proj-feat-search, making checkpoint commits

# 3. before opening a PR: rebase onto main
cd ../proj-feat-search
git fetch origin
git rebase origin/main

# 4. push and open a PR
git push -u origin feat/search
gh pr create

# 5. review the diff (not the summary), let CI run

# 6. squash-merge via the PR UI

# 7. clean up
cd ../proj
git worktree remove ../proj-feat-search
git branch -d feat/search
```

That's the whole loop. Short-lived isolated branches, checkpoint commits during development, rebase to update, squash to land.

## Further reading

- [Pro Git — Git Worktree](https://git-scm.com/docs/git-worktree)
- [Trunk-Based Development](https://trunkbaseddevelopment.com/)
- [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow)
