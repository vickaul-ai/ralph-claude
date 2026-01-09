---
name: ralph-init
description: "Initialize Ralph in the current project. Use after /prd and /ralph to set up the autonomous loop scripts. Triggers on: initialize ralph, setup ralph, ralph init, add ralph to project."
---

# Ralph Project Initializer

Set up Ralph scripts in the current project directory so you can run the autonomous agent loop.

---

## The Job

1. Create the `scripts/ralph/` directory in the current project
2. Copy `ralph.sh` and `prompt.md` from `~/Documents/Projects/ralph-claude/`
3. Make `ralph.sh` executable
4. Confirm setup is complete

---

## Prerequisites

Before running `/ralph-init`, you should have:

1. Generated a PRD with `/prd` (saved to `tasks/prd-*.md`)
2. Converted it to prd.json with `/ralph` (creates `scripts/ralph/prd.json`)

---

## What Gets Created

```
your-project/
  scripts/
    ralph/
      ralph.sh      # The iteration loop script
      prompt.md     # Instructions for each Claude Code iteration
      prd.json      # Your user stories (created by /ralph)
      progress.txt  # Created automatically on first run
```

---

## Steps to Execute

When this skill is invoked:

```bash
# 1. Create the scripts/ralph directory
mkdir -p scripts/ralph

# 2. Copy the core files
cp ~/Documents/Projects/ralph-claude/ralph.sh scripts/ralph/
cp ~/Documents/Projects/ralph-claude/prompt.md scripts/ralph/

# 3. Make ralph.sh executable
chmod +x scripts/ralph/ralph.sh
```

---

## After Initialization

Tell the user:

1. Move their `prd.json` to `scripts/ralph/` if it isn't there already
2. Run the autonomous loop: `./scripts/ralph/ralph.sh 10`
3. Monitor progress in `scripts/ralph/progress.txt`

---

## Example Output

After successful initialization, respond:

```
Ralph initialized in scripts/ralph/

Next steps:
1. Ensure prd.json is in scripts/ralph/
2. Run: ./scripts/ralph/ralph.sh 10

The script will iterate until all user stories pass or max iterations reached.
```

---

## Notes

- The ralph.sh script uses Claude Code CLI (`claude --dangerously-skip-permissions`)
- Each iteration spawns a fresh Claude Code instance
- Progress and learnings are persisted in `progress.txt`
- Completed stories are tracked in `prd.json` with `passes: true`
