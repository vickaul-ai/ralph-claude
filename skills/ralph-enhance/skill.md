---
name: ralph-enhance
description: "Analyze a feature request and advise on the best Ralph approach for extending an existing Ralph-built application. Use when planning new features, considering enhancements, or deciding how to add functionality. Triggers on: ralph enhance, extend ralph app, add feature to ralph project, how should I add, plan enhancement, enhance this app."
---

# Ralph Enhancement Advisor

Analyze a feature request and recommend the optimal approach for extending a Ralph-built application.

---

## The Job

1. Read the feature request (from user message or feature-[name].md file)
2. Study the existing codebase:
   - Read `scripts/ralph/progress.txt` for patterns and learnings
   - Read `scripts/ralph/prd.json` for current state and completed stories
   - Explore relevant code areas
3. Provide a structured recommendation

**Important:** Do NOT start implementing. Just analyze and advise.

---

## Analysis Steps

### Step 1: Gather Context

```bash
# Read progress.txt for patterns
cat scripts/ralph/progress.txt | head -100

# Read prd.json for current state
cat scripts/ralph/prd.json | jq '.userStories | length' # Count stories
cat scripts/ralph/prd.json | jq '.userStories[] | select(.passes == false)' # Pending stories

# Check if there's an existing archive
ls scripts/ralph/archive/ 2>/dev/null
```

### Step 2: Assess Feature Size

Count the likely user stories:
- **Schema changes?** → 1-2 stories per table
- **Backend logic?** → 1-2 stories per endpoint/action
- **UI components?** → 1-2 stories per component
- **Integration?** → 1 story per external service

### Step 3: Check for Conflicts

Look for:
- Files that would need modification vs. addition
- Components that might break with changes
- Database schema dependencies

---

## Decision Framework

| Stories | Complexity | Existing PRD Status | Recommendation |
|---------|------------|---------------------|----------------|
| 15+ | Any | Any | **New PRD** |
| 5-15 | Medium-High | Any | **New PRD** |
| 5-15 | Low | All passing | Could append, prefer **New PRD** |
| 1-4 | Any | Has pending stories | **Wait or New PRD** |
| 1-4 | Related | All passing | **Append stories** |
| 1-4 | Unrelated | All passing | **New PRD** (clean separation) |
| <5 small changes | Low | Any | **Direct implementation** |

---

## Output Format

Always provide the recommendation in this structure:

```markdown
## Ralph Enhancement Analysis: [Feature Name]

### Recommendation: [New PRD | Append Stories | Direct Implementation | Wait]

### Reasoning
- [Why this approach based on the analysis]
- [Reference to existing patterns or code]

### Suggested Stories
If New PRD or Append:
1. **US-XXX: [Title]** - Size: [Small/Medium]
   - Key acceptance criteria
   - Patterns to reference from progress.txt

2. **US-XXX: [Title]** - Size: [Small/Medium]
   - Key acceptance criteria

### Dependencies
- Requires: [existing components, tables, or files]
- Order: [Schema → Backend → UI or other dependency chain]

### Patterns to Reference
From progress.txt Codebase Patterns section:
- [Pattern 1 that applies]
- [Pattern 2 that applies]

### Risk Areas
- [Area]: [Risk and mitigation]

### Next Steps
1. [First action - e.g., "Run /prd 'feature name'"]
2. [Second action - e.g., "Convert with /ralph"]
3. [Third action - e.g., "Run ./scripts/ralph/ralph.sh 15"]
```

---

## Examples

### Example 1: New Major Feature

**User:** "I want to add Notion export to the app"

**Analysis:**
- Requires OAuth integration (schema + backend)
- Requires export logic (backend)
- Requires UI buttons (frontend)
- ~8-10 stories

**Output:**
```markdown
## Ralph Enhancement Analysis: Notion Export

### Recommendation: New PRD

### Reasoning
This is a significant feature requiring OAuth, API integration, and UI changes.
Approximately 8-10 stories across schema, backend, and frontend layers.

### Suggested Stories
1. **US-001: Add Notion OAuth credentials table** - Size: Small
   - Add notion_connections table (user_id, access_token, workspace_id)
   - Migration runs successfully
   - Typecheck passes

2. **US-002: Create Notion OAuth flow** - Size: Medium
   - /api/notion/auth endpoint redirects to Notion OAuth
   - Callback stores tokens in database
   - Follow OAuth pattern from existing auth code

[... continue for all stories]

### Next Steps
1. Run `/prd "Notion export integration"`
2. Convert with `/ralph tasks/prd-notion-export.md`
3. Run `./scripts/ralph/ralph.sh 20`
```

### Example 2: Small Related Addition

**User:** "Add a button to copy the research report URL to clipboard"

**Analysis:**
- Single UI change
- Related to existing export functionality (US-024)
- 1 story

**Output:**
```markdown
## Ralph Enhancement Analysis: Copy URL Button

### Recommendation: Append Stories

### Reasoning
Single small change, closely related to existing export functionality in US-024.
Current prd.json has all stories passing. Safe to append.

### Suggested Stories
1. **US-033: Add copy URL button to research tab** - Size: Small
   - Button appears next to existing Download button
   - Uses existing handleCopy pattern from US-024
   - Shows toast notification on success
   - Typecheck passes
   - Verify in browser using claude-in-chrome MCP tools

### Patterns to Reference
- Copy pattern from US-024: navigator.clipboard.writeText()
- Toast notification pattern from submission detail page

### Next Steps
1. Add story to scripts/ralph/prd.json with passes: false
2. Run `./scripts/ralph/ralph.sh 3`
```

### Example 3: Direct Implementation

**User:** "Change the button color from blue to green"

**Output:**
```markdown
## Ralph Enhancement Analysis: Button Color Change

### Recommendation: Direct Implementation

### Reasoning
Single CSS change, no logic involved. Faster to implement directly than
setting up a Ralph run.

### Next Steps
1. Find the button in the relevant component
2. Change the Tailwind class (e.g., bg-blue-600 → bg-green-600)
3. Verify visually
4. Commit directly
```

---

## Checklist Before Advising

- [ ] Read progress.txt Codebase Patterns section
- [ ] Checked prd.json for pending stories
- [ ] Assessed feature complexity (schema/backend/UI)
- [ ] Counted likely stories
- [ ] Checked for potential conflicts
- [ ] Referenced existing patterns in recommendation
- [ ] Provided clear next steps
