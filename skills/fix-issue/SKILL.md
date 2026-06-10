---
name: fix-issue
description: Use when fixing a bug, UI issue, data problem, or unexpected behaviour. Covers branch setup, DB investigation, root cause analysis, fix, verification, RCA doc, and PR.
---

# Fix Issue

## Overview

Follow this workflow for every bug fix. The goal is: understand before touching, verify before closing, document always.

## Workflow

```
1. Branch  →  2. Investigate  →  3. Fix  →  4. Verify  →  5. Document  →  6. PR
```

---

### 1. Branch

```bash
git checkout main && git pull
git checkout -b fix/<short-kebab-description>
```

---

### 2. Investigate — find root cause first

Never write a fix before the root cause is confirmed. Follow the steps below to trace systematically.

**Trace the full data path:**
- Frontend: which component, which API call, what does the response actually contain?
- Backend: controller → service/model → DB query → what does the query return?
- Schema: check your schema definitions — does the column actually exist in the DB?

**Read-only DB access** when you need to inspect live data directly:

```bash
# Connect with read-only flag to prevent accidental writes
PGOPTIONS="-c default_transaction_read_only=on" \
  psql -h <DB_HOST> -p <DB_PORT> -U <DB_USER> -d <DB_NAME>
```

Useful queries:
```sql
-- Inspect a table's actual columns
\d <table_name>

-- Sample rows
SELECT * FROM <table_name> LIMIT 5;

-- Check what a query actually returns
EXPLAIN ANALYZE SELECT ...;
```

> Never run INSERT/UPDATE/DELETE — use the read-only flag to prevent accidents.

---

### 3. Fix

**After every code change, run lint and type checks:**
```bash
# Adapt to your project's lint setup:
npm run lint
npx tsc --noEmit
```

**Run the relevant tests:**
```bash
npm test -- --testPathPattern=<module>
```

Add a test if the bug had no coverage.

---

### 4. Verify

Run the app and visually confirm the fix at the UI surface. Don't claim PASS from tests alone — run the actual page or endpoint and observe the corrected behaviour with your own eyes.

---

### 5. Document — RCA in your issues directory

Every fix gets a doc. Find the next issue number:

```bash
ls docs/issues/ | sort | tail -1
```

Create `docs/issues/<PREFIX>-<NNN>-<short-title>.md` using this template:

```markdown
# <PREFIX>-NNN: <Title>

**Date**: YYYY-MM-DD  
**Severity**: Low | Medium | High | Critical  
**Status**: Resolved  

---

## Problem

One paragraph: what the user saw, on which screen, under what conditions.

## Root Cause

What was actually wrong and why. Be specific — file path, line number, field name.

## Fix

What changed and why that resolves the root cause. Link to PR/commit.

## Prevention

What would catch this earlier next time (test, lint rule, schema constraint, etc.).
```

---

### 6. PR

```bash
git add <files>       # never git add -A or git add .
git commit -m "fix(<scope>): <what and why>"
git push -u origin fix/<branch>
gh pr create --title "fix(<scope>): ..." --body "..."
```

Merge only when:
- `gh pr view --json mergeable` → `MERGEABLE`
- `mergeStateStatus` → `CLEAN`
- No failing CI checks

```bash
gh pr merge <number> --squash --delete-branch
git checkout main && git pull
```

---

## Quick Reference

| Step | What to do |
|------|------------|
| Read-only DB | Connect with `PGOPTIONS="-c default_transaction_read_only=on"` |
| Lint | Run your project's lint script |
| Type check | `npx tsc --noEmit` |
| Run tests | `npm test -- --testPathPattern=<module>` |
| Issue docs | `docs/issues/<PREFIX>-NNN-title.md` |

## Things That Bite

- **"Unknown" or missing data in UI** → check whether the field actually exists on the returned object; don't assume joins add the field you expect
- **ORM join queries** → always verify how your ORM handles null from left joins; test with rows that have no match
- **Migration tracking can be inaccurate** → after running migrations, query the DB directly to confirm the column exists
- **Commits without user confirmation** → never push to remote without the user confirming the fix first; ask if unclear
