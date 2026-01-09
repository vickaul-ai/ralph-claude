# Getting Started with Ralph

A complete guide to autonomous AI development using Ralph with Claude Code.

---

## What is Ralph?

Ralph is an **autonomous AI agent loop** that builds features for you. You describe what you want in plain English, Ralph breaks it into small tasks, then executes them one by one—writing code, running tests, committing changes—until the feature is complete.

Think of it as giving an AI developer a to-do list and letting them work through it independently.

**How it works:**
1. You write a Product Requirements Document (PRD)
2. Ralph converts it into small, executable user stories
3. Ralph loops through each story: implement → test → commit → repeat
4. You review the completed feature

---

## Prerequisites

Before starting, ensure you have:

| Requirement | How to Get It |
|-------------|---------------|
| **Claude Code CLI** | Install from [claude.ai/code](https://claude.ai/code) and authenticate |
| **jq** | `brew install jq` (macOS) or `apt install jq` (Linux) |
| **Git repository** | Your project must be a git repo |
| **Node.js** (optional) | Only needed if you want to run the flowchart locally |

Verify your setup:
```bash
claude --version    # Should show Claude Code version
jq --version        # Should show jq version
git status          # Should show you're in a git repo
```

---

## One-Time Setup: Install Ralph Skills

Ralph uses three Claude Code skills. Install them once and they're available in all your projects:

```bash
# Clone the ralph-claude repo (or download it)
git clone https://github.com/vickaul-ai/ralph-claude.git ~/ralph-claude

# Copy skills to your Claude Code config
cp -r ~/ralph-claude/skills/prd ~/.claude/skills/
cp -r ~/ralph-claude/skills/ralph ~/.claude/skills/
cp -r ~/ralph-claude/skills/ralph-init ~/.claude/skills/
```

**What each skill does:**
- `/prd` — Generates a detailed PRD from your idea through an interactive interview
- `/ralph` — Converts a PRD into the JSON format Ralph needs
- `/ralph-init` — Sets up Ralph scripts in your current project

---

## Example Walkthrough: Building ProjectX

Let's walk through a complete example. Imagine you're building **ProjectX**, a task management app, and you want to add a priority feature to tasks.

### Step 1: Start Claude Code in Your Project

```bash
cd ~/projects/projectx
claude
```

### Step 2: Generate a PRD with `/prd`

In Claude Code, run:
```
/prd Add priority levels to tasks
```

Claude will interview you with questions like:
- What priority levels do you want? (High, Medium, Low)
- Should there be a default priority?
- How should priorities be displayed in the UI?
- Should users be able to filter by priority?

Answer the questions. When complete, Claude saves a detailed PRD to:
```
tasks/prd-task-priorities.md
```

**Example PRD output:**
```markdown
# Task Priorities Feature

## Overview
Add priority levels (High, Medium, Low) to tasks with visual indicators and filtering.

## User Stories
1. As a user, I can set a priority when creating a task
2. As a user, I can see priority indicators on tasks
3. As a user, I can filter tasks by priority
4. As a user, I can change a task's priority

## Acceptance Criteria
- Priority stored in database with default "Medium"
- Color-coded badges: Red (High), Yellow (Medium), Green (Low)
- Filter dropdown in task list header
- Priority editable inline
...
```

### Step 3: Convert PRD to Ralph Format with `/ralph`

```
/ralph tasks/prd-task-priorities.md
```

This creates `prd.json` with your feature broken into small, executable stories:

```json
{
  "feature": "Task Priorities",
  "branchName": "feature/task-priorities",
  "userStories": [
    {
      "id": "US-001",
      "title": "Add priority column to database",
      "priority": 1,
      "acceptanceCriteria": [
        "Add priority enum column to tasks table",
        "Create and run migration",
        "Default value is 'medium'",
        "Typecheck passes"
      ],
      "passes": false
    },
    {
      "id": "US-002",
      "title": "Display priority badge on tasks",
      "priority": 2,
      "acceptanceCriteria": [
        "Create PriorityBadge component",
        "Show badge on TaskCard component",
        "Colors: red=high, yellow=medium, green=low",
        "Verify in browser using claude-in-chrome MCP tools"
      ],
      "passes": false
    },
    {
      "id": "US-003",
      "title": "Add priority filter dropdown",
      "priority": 3,
      "acceptanceCriteria": [
        "Add filter dropdown to TaskList header",
        "Filter updates task list in real-time",
        "Verify in browser using claude-in-chrome MCP tools"
      ],
      "passes": false
    }
  ]
}
```

**Key points about user stories:**
- Each story should be completable in **one Claude Code session** (small scope)
- Stories are ordered by `priority` (1 = first)
- `passes: false` means not yet completed
- Acceptance criteria are specific and testable

### Step 4: Initialize Ralph in Your Project with `/ralph-init`

```
/ralph-init
```

This creates the Ralph scripts in your project:
```
projectx/
  scripts/
    ralph/
      ralph.sh      # The main loop script
      prompt.md     # Instructions for each iteration
```

Move your prd.json to the ralph folder:
```bash
mv prd.json scripts/ralph/
```

### Step 5: Run Ralph

Exit Claude Code (Ctrl+C) and run Ralph from your terminal:

```bash
./scripts/ralph/ralph.sh 15
```

The number `15` is the maximum iterations. Ralph will:

1. **Create a feature branch** (`feature/task-priorities`)
2. **Pick the first incomplete story** (US-001)
3. **Implement it** (write code, run migrations)
4. **Run quality checks** (typecheck, tests)
5. **Commit if passing**
6. **Update prd.json** (`passes: true`)
7. **Log learnings** to `progress.txt`
8. **Move to next story**
9. **Repeat until all stories pass**

### Step 6: Monitor Progress

While Ralph runs, you can monitor in another terminal:

```bash
# See which stories are complete
cat scripts/ralph/prd.json | jq '.userStories[] | {id, title, passes}'

# See learnings and context from iterations
cat scripts/ralph/progress.txt

# See commits made
git log --oneline -10
```

**Example progress.txt:**
```
=== Iteration 1 ===
Completed: US-001 - Add priority column to database
Learnings: This codebase uses Drizzle ORM. Migrations are in src/db/migrations.
Updated AGENTS.md with migration patterns.

=== Iteration 2 ===
Completed: US-002 - Display priority badge on tasks
Learnings: Components are in src/components. Uses Tailwind for styling.
Badge component created at src/components/PriorityBadge.tsx
```

### Step 7: Review and Merge

When Ralph finishes (all stories `passes: true`), review the changes:

```bash
# See all changes
git diff main

# Run your full test suite
npm test

# If everything looks good, merge
git checkout main
git merge feature/task-priorities
```

---

## Sharing Progress: The Interactive Flowchart

Ralph includes an interactive flowchart that explains how it works. This is useful for:
- **Onboarding team members** to understand the Ralph workflow
- **Presentations** explaining autonomous AI development
- **Documentation** for your team's processes

### View the Live Flowchart

**[https://vickaul-ai.github.io/ralph-claude/](https://vickaul-ai.github.io/ralph-claude/)**

Click through each step to see the flow with animations.

### Run Locally for Presentations

```bash
cd ~/ralph-claude/flowchart
npm install
npm run dev
```

Opens at `http://localhost:5173`. Use Previous/Next buttons to walk through each step.

### Customize for Your Team

Fork the repo and modify the flowchart for your specific workflow:

1. Edit `flowchart/src/App.tsx` to change steps, labels, or add notes
2. Rebuild: `npm run build`
3. Deploy: `npm run deploy` (publishes to your GitHub Pages)

---

## Best Practices

### Writing Good User Stories

**Right-sized (completable in one session):**
- Add a database column and migration
- Create a single UI component
- Add a filter to an existing list
- Update a server action with new logic

**Too large (split these up):**
- "Build the dashboard" → Split into: header, sidebar, main content, each widget
- "Add authentication" → Split into: login form, signup form, session handling, protected routes
- "Refactor the API" → Split into specific endpoints or patterns

### Acceptance Criteria Tips

Make criteria **specific and verifiable**:

```json
// Good
"acceptanceCriteria": [
  "Add priority column to tasks table",
  "Column type is enum: 'high', 'medium', 'low'",
  "Default value is 'medium'",
  "Migration runs without errors",
  "npm run typecheck passes"
]

// Too vague
"acceptanceCriteria": [
  "Priority should work",
  "Database updated"
]
```

### Include Browser Verification for UI

For any frontend work, add this criterion:
```json
"Verify in browser using claude-in-chrome MCP tools"
```

Ralph will actually navigate to the page and verify the UI works.

### Keep Your Feedback Loops

Ralph only works if there are automated checks:
- **TypeScript** catches type errors
- **Tests** verify behavior
- **Linting** catches style issues

If these aren't set up, Ralph can't verify its work.

---

## Troubleshooting

### Ralph stops before completing all stories

Check `progress.txt` for errors. Common issues:
- Tests failing (fix the test or the code)
- TypeScript errors (check the types)
- Story too large (split into smaller stories)

### Stories keep failing the same way

Add context to `AGENTS.md` in your project root:
```markdown
# Agent Instructions

## Patterns
- This project uses Drizzle ORM, not Prisma
- Run `npm run db:push` after schema changes
- Components go in src/components, not src/ui
```

Claude Code reads this file automatically.

### Can I pause and resume?

Yes. Ralph tracks progress in `prd.json`. Just run `./scripts/ralph/ralph.sh` again and it picks up where it left off.

### Can I manually complete a story?

Yes. Edit `prd.json` and set `"passes": true` for any story you complete yourself.

---

## Command Reference

| Command | What It Does |
|---------|--------------|
| `/prd [idea]` | Generate a PRD through interactive interview |
| `/ralph [prd-file]` | Convert PRD markdown to prd.json |
| `/ralph-init` | Set up Ralph scripts in current project |
| `./scripts/ralph/ralph.sh [n]` | Run Ralph for up to n iterations |
| `cat scripts/ralph/prd.json \| jq '.userStories[]'` | Check story status |
| `cat scripts/ralph/progress.txt` | View iteration logs |

---

## Next Steps

1. **Try it on a small feature first** — Get comfortable with the workflow
2. **Refine your PRD skills** — Better PRDs = better results
3. **Customize prompt.md** — Add project-specific instructions
4. **Share the flowchart** — Onboard your team

Questions? Check the [README](../README.md) or the [original Ralph article](https://ghuntley.com/ralph/).
