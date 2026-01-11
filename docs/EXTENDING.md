# Extending Ralph-Built Applications

This guide covers how to add features to an application that was built using the Ralph autonomous agent loop.

## Quick Reference

| Situation | Action | Command |
|-----------|--------|---------|
| **New major feature** (5+ stories) | Create new PRD | `/prd` → `/ralph` → `./ralph.sh 20` |
| **Small addition** (1-4 related stories) | Append to prd.json | Edit prd.json → `./ralph.sh 5` |
| **Bug fix** | Direct fix | No PRD needed |
| **Quick polish** (<5 small changes) | Direct implementation | Faster than Ralph |

---

## Why Ralph Extensions Work Differently

### Each Iteration = Fresh Context

Ralph spawns a **new Claude Code instance** per iteration with no memory. Memory persists only via:
- Git history
- `progress.txt` (learnings)
- `prd.json` (task status)
- `AGENTS.md` (project patterns)

### Automatic Archiving

Your `ralph.sh` archives automatically when `branchName` changes:
- Previous run saved to `archive/YYYY-MM-DD-feature-name/`
- Fresh `progress.txt` created for new runs

---

## Option A: New Feature PRD (Recommended)

For features requiring 5+ user stories:

```bash
# 1. Create new PRD
/prd "Add Notion export feature"

# 2. Answer clarifying questions
# PRD saved to tasks/prd-notion-export.md

# 3. Convert to Ralph format
/ralph tasks/prd-notion-export.md
# Creates prd.json with branchName: "ralph/notion-export"
# Old prd.json auto-archived

# 4. Run the loop
./scripts/ralph/ralph.sh 20
```

---

## Option B: Append Stories

For 1-4 small, related additions:

```json
// Add to existing prd.json
{
  "id": "US-033",
  "title": "Add copy URL button",
  "description": "As a user, I want to copy the submission URL.",
  "acceptanceCriteria": [
    "Button appears next to Download",
    "Follow copy pattern from US-024",
    "Typecheck passes"
  ],
  "priority": 33,
  "passes": false,
  "notes": ""
}
```

Then run: `./scripts/ralph/ralph.sh 5`

**Warning:** Don't modify stories where `passes: true`. Ralph may re-run them.

---

## Option C: Multi-Phase Approach

For large features, split into phases:

```bash
# Phase 1: Schema
/prd "Notifications - schema"
./scripts/ralph/ralph.sh 10
# Verify, merge to main

# Phase 2: Backend
/prd "Notifications - backend"
./scripts/ralph/ralph.sh 15

# Phase 3: Frontend
/prd "Notifications - UI"
./scripts/ralph/ralph.sh 20
```

---

## Writing Effective Stories

### The Number One Rule

**Each story must be completable in ONE iteration.**

If a story is too big, Ralph runs out of context and produces broken code.

### Story Sizing

| Right-sized | Too big |
|-------------|---------|
| Add a database column | Build entire dashboard |
| Add a UI component | Add authentication |
| Update a server action | Refactor the API |

**Rule:** If you can't describe it in 2-3 sentences, split it.

### Story Ordering

Dependencies first:
1. **Schema/Database** (migrations)
2. **Backend logic** (server actions)
3. **UI components** (use the backend)

### Acceptance Criteria

| Good (Verifiable) | Bad (Vague) |
|-------------------|-------------|
| "Add status column with default 'pending'" | "Works correctly" |
| "Typecheck passes" | "Good UX" |
| "Verify in browser using claude-in-chrome" | "Handles edge cases" |

### Reference Existing Patterns

From your `progress.txt`:

```json
{
  "acceptanceCriteria": [
    "Follow server action pattern from src/app/actions/",
    "Use existing ConfirmDialog component",
    "Typecheck passes"
  ]
}
```

---

## Avoiding Conflicts

### Stories Must Be Additive

| Good | Bad |
|------|-----|
| "Add button to page" | "Redesign page layout" |
| "Add option to dropdown" | "Replace dropdown" |

### Use Feature Branches

Each PRD has a `branchName`:

```json
{ "branchName": "ralph/notion-export" }
```

After completion: Review → Merge to main → Start new PRD

### When Things Go Wrong

Add "Signs" (constraints) to stories:
- "Do NOT modify component X"
- "Use existing Y instead of creating new"
- "Follow pattern from Z"

---

## Using /ralph-enhance

For guidance on how to approach a new feature:

```
/ralph-enhance "I want to add Notion export"
```

This skill will:
1. Read your progress.txt for patterns
2. Check prd.json for current state
3. Recommend: New PRD vs Append vs Direct
4. Suggest story breakdown with sizes
5. Identify patterns to reference
6. Warn about risk areas

---

## Decision Flowchart

```
Feature Request
      │
      ▼
How many stories?
      │
      ├── 15+ stories ──────────────────► New PRD
      │
      ├── 5-15 stories ─────────────────► New PRD (usually)
      │
      ├── 1-4 stories
      │        │
      │        ▼
      │   Related to existing feature?
      │        │
      │        ├── Yes ──────────────────► Append stories
      │        └── No ───────────────────► New PRD
      │
      └── <5 small changes ─────────────► Direct implementation
```

---

## Further Reading

- [Geoffrey Huntley's Ralph article](https://ghuntley.com/ralph/)
- [Cursed: 3-month Ralph project](https://ghuntley.com/cursed/)
- [Brief History of Ralph](https://www.humanlayer.dev/blog/brief-history-of-ralph)
