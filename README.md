# Ralph

![Ralph](ralph.webp)

Ralph is an autonomous AI agent loop that runs [Claude Code](https://claude.ai/code) repeatedly until all PRD items are complete. Each iteration is a fresh Claude Code instance with clean context. Memory persists via git history, `progress.txt`, and `prd.json`.

Based on [Geoffrey Huntley's Ralph pattern](https://ghuntley.com/ralph/).

**[Getting Started Guide](docs/GETTING_STARTED.md)** — Complete walkthrough with a practical example

**[Extending Guide](docs/EXTENDING.md)** — How to add features to existing Ralph-built apps

## Prerequisites

- [Claude Code CLI](https://claude.ai/code) installed and authenticated
- `jq` installed (`brew install jq` on macOS)
- A git repository for your project

## Setup

### Option 1: Copy to your project

Copy the ralph files into your project:

```bash
# From your project root
mkdir -p scripts/ralph
cp /path/to/ralph-claude/ralph.sh scripts/ralph/
cp /path/to/ralph-claude/prompt.md scripts/ralph/
chmod +x scripts/ralph/ralph.sh
```

### Option 2: Install skills globally

Copy the skills to your Claude Code config for use across all projects:

```bash
cp -r skills/prd ~/.claude/skills/
cp -r skills/ralph ~/.claude/skills/
cp -r skills/ralph-init ~/.claude/skills/
cp -r skills/ralph-enhance ~/.claude/skills/
```

Then use `/ralph-init` in any project to set up the scripts automatically.

Available skills:
- `/prd` - Generate a Product Requirements Document
- `/ralph` - Convert PRD to prd.json format
- `/ralph-init` - Initialize Ralph in a project
- `/ralph-enhance` - Get advice on extending an existing Ralph-built app

## Workflow

### 1. Create a PRD

Use the PRD skill to generate a detailed requirements document:

```
/prd [your feature description]
```

Answer the clarifying questions. The skill saves output to `tasks/prd-[feature-name].md`.

### 2. Convert PRD to Ralph format

Use the Ralph skill to convert the markdown PRD to JSON:

```
/ralph tasks/prd-[feature-name].md
```

This creates `prd.json` with user stories structured for autonomous execution.

### 3. Initialize Ralph in your project

```
/ralph-init
```

This copies `ralph.sh` and `prompt.md` to `scripts/ralph/`.

### 4. Run Ralph

```bash
./scripts/ralph/ralph.sh [max_iterations]
```

Default is 10 iterations.

Ralph will:
1. Create a feature branch (from PRD `branchName`)
2. Pick the highest priority story where `passes: false`
3. Implement that single story
4. Run quality checks (typecheck, tests)
5. Commit if checks pass
6. Update `prd.json` to mark story as `passes: true`
7. Append learnings to `progress.txt`
8. Repeat until all stories pass or max iterations reached

## Key Files

| File | Purpose |
|------|---------|
| `ralph.sh` | The bash loop that spawns fresh Claude Code instances |
| `prompt.md` | Instructions given to each Claude Code instance |
| `prd.json` | User stories with `passes` status (the task list) |
| `prd.json.example` | Example PRD format for reference |
| `progress.txt` | Append-only learnings for future iterations |
| `skills/prd/` | Skill for generating PRDs |
| `skills/ralph/` | Skill for converting PRDs to JSON |
| `skills/ralph-init/` | Skill for initializing Ralph in a project |
| `skills/ralph-enhance/` | Skill for advising on extensions |
| `docs/EXTENDING.md` | Guide for extending Ralph-built apps |
| `flowchart/` | Interactive visualization of how Ralph works |

## Flowchart

[![Ralph Flowchart](ralph-flowchart.png)](https://vickaul-ai.github.io/ralph-claude/)

**[View Interactive Flowchart](https://vickaul-ai.github.io/ralph-claude/)** - Click through to see each step with animations.

The `flowchart/` directory contains the source code. To run locally:

```bash
cd flowchart
npm install
npm run dev
```

## Critical Concepts

### Each Iteration = Fresh Context

Each iteration spawns a **new Claude Code instance** with clean context. The only memory between iterations is:
- Git history (commits from previous iterations)
- `progress.txt` (learnings and context)
- `prd.json` (which stories are done)

### Small Tasks

Each PRD item should be small enough to complete in one context window. If a task is too big, the LLM runs out of context before finishing and produces poor code.

Right-sized stories:
- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

Too big (split these):
- "Build the entire dashboard"
- "Add authentication"
- "Refactor the API"

### AGENTS.md Updates Are Critical

After each iteration, Ralph updates the relevant `AGENTS.md` files with learnings. This is key because Claude Code automatically reads these files, so future iterations (and future human developers) benefit from discovered patterns, gotchas, and conventions.

Examples of what to add to AGENTS.md:
- Patterns discovered ("this codebase uses X for Y")
- Gotchas ("do not forget to update Z when changing W")
- Useful context ("the settings panel is in component X")

### Feedback Loops

Ralph only works if there are feedback loops:
- Typecheck catches type errors
- Tests verify behavior
- CI must stay green (broken code compounds across iterations)

### Browser Verification for UI Stories

Frontend stories must include "Verify in browser using claude-in-chrome MCP tools" in acceptance criteria. Ralph will use the claude-in-chrome MCP tools to navigate to the page, interact with the UI, and confirm changes work.

### Stop Condition

When all stories have `passes: true`, Ralph outputs `<promise>COMPLETE</promise>` and the loop exits.

## Philosophy: Tuning Ralph

Ralph is like a guitar that needs tuning. When an iteration fails, resist the urge to blame the tools. Instead:

1. **Analyze the failure** - What went wrong and why?
2. **Add "signs"** - Update prompt.md with clearer instructions
3. **Refine specs** - Vague acceptance criteria cause vague implementations
4. **Trust eventual consistency** - Wrong paths get corrected through iteration

> "Building software with Ralph requires a great deal of faith and a belief in eventual consistency." — Geoffrey Huntley

## Common Pitfalls

| Pitfall | Symptom | Solution |
|---------|---------|----------|
| **Overbaking** | Bizarre emergent behaviors (features you didn't ask for) | Review and merge more frequently |
| **One-shotting** | Broken code, incomplete features | Split into smaller stories |
| **Context pollution** | Unrelated content in output | Fresh context per iteration (Ralph does this) |
| **Vague specs** | Inconsistent implementations | Add specific acceptance criteria |
| **Missing feedback loops** | Can't tell if code works | Add typecheck, tests, browser verification |

## When Ralph Gets Stuck

Ralph tracks attempt numbers in progress.txt (e.g., "US-001 (Attempt 3)"). If you see high attempt counts or Ralph hits max iterations:

### 1. Diagnose the Problem

```bash
# Check which stories are incomplete
cat scripts/ralph/prd.json | jq '.userStories[] | select(.passes == false)'

# Look for high attempt counts
grep -E "Attempt [3-9]|Attempt [0-9]{2}" scripts/ralph/progress.txt

# Read iteration logs for patterns
cat scripts/ralph/progress.txt
```

### 2. Common Fixes

| Problem | Solution |
|---------|----------|
| Same error repeating (Attempt 3+) | Add explicit instruction to prompt.md |
| Story too large | Split into smaller stories |
| Missing context | Add to AGENTS.md or progress.txt Codebase Patterns |
| Spec ambiguity | Rewrite acceptance criteria |
| Technical blocker | Mark story for manual implementation |

### 3. Recovery Options

- **Tune prompt.md** - Add "signs" based on failure patterns
- **Refine the story** - Edit prd.json with clearer acceptance criteria
- **Split the story** - Break into 2-3 smaller pieces
- **Revert and pivot** - `git revert` to good state, try different approach
- **Manual takeover** - Some stories need human implementation

### 4. Resume

After making adjustments:

```bash
./scripts/ralph/ralph.sh 5  # Continue with fresh iterations
```

## Debugging

Check current state:

```bash
# See which stories are done
cat prd.json | jq '.userStories[] | {id, title, passes}'

# See learnings from previous iterations
cat progress.txt

# Check git history
git log --oneline -10
```

## Customizing prompt.md

Edit `prompt.md` to customize Ralph's behavior for your project:
- Add project-specific quality check commands
- Include codebase conventions
- Add common gotchas for your stack

## Archiving

Ralph automatically archives previous runs when you start a new feature (different `branchName`). Archives are saved to `archive/YYYY-MM-DD-feature-name/`.

## Extending Ralph-Built Apps

After completing the initial build, use this decision framework for adding features:

| Situation | Action |
|-----------|--------|
| **New major feature** (5+ stories) | Create new PRD: `/prd` → `/ralph` → `./ralph.sh` |
| **Small addition** (1-4 related stories) | Append to existing prd.json |
| **Bug fix** | Direct fix, no PRD needed |
| **Quick polish** (<5 small changes) | Direct implementation |

Use `/ralph-enhance` to get advice on the best approach for your specific feature.

**[Full Extending Guide →](docs/EXTENDING.md)**

## References

### Core

- [Geoffrey Huntley's Ralph article](https://ghuntley.com/ralph/) - Original technique
- [CURSED: 3-month Ralph project](https://ghuntley.com/cursed/) - Extreme test case

### Deep Dives

- [How to Build a Coding Agent Workshop](https://ghuntley.com/agent/) - Comprehensive implementation guide
- [Anthropic: Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) - Feature list pattern
- [Anthropic: Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices) - Context optimization

### Community

- [Brief History of Ralph](https://www.humanlayer.dev/blog/brief-history-of-ralph) - Timeline and lessons learned
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)

### Implementations & Extensions

- [frankbria/ralph-claude-code](https://github.com/frankbria/ralph-claude-code) - Circuit breaker with advanced error detection, dual-condition exit gates
- [thecgaigroup/ralph-cc-loop](https://github.com/thecgaigroup/ralph-cc-loop) - PRD-based task tracking with `dependsOn` field

### Stuck Detection & Governance

- [Supervising Ralph: Principal Skinner](https://securetrajectories.substack.com/p/ralph-wiggum-principal-skinner-agent-reliability) - Behavioral circuit breakers, infrastructure-level safety
- [Why AI Agents Get Stuck in Loops](https://www.fixbrokenaiapps.com/blog/ai-agents-infinite-loops) - Loop guardrails framework, repetitive output detection
- [HN: Controller-enforced alternative to Ralph](https://news.ycombinator.com/item?id=46651646) - Tests as acceptance gate, git as safety harness
