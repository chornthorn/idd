---
name: intent-changes
description: Manage structured change proposals for design documents with PR-like review experience. Use /intent-changes start <file> to begin, /intent-changes propose to suggest changes, /intent-changes accept/reject to decide, /intent-changes finalize to apply.
---

# Intent Changes

Structured change proposal and collaborative review tool for design documents.

## Core Concepts

### Change Proposal

Each change is an independent proposal:

| Field | Description |
|-------|-------------|
| **ID** | Unique identifier (C001, C002...) |
| **Type** | ADD / MODIFY / REPLACE / DELETE |
| **Status** | PENDING / ACCEPTED / REJECTED |
| **Target** | Change location |
| **Before/After** | Change content |
| **Decision** | Decision record (reviewer, timestamp, comment) |

### Status Flow

```
PENDING ──accept──> ACCEPTED ──finalize──> Applied
    │
    └──reject──> REJECTED
```

## Commands

| Command | Description |
|---------|-------------|
| `/intent-changes start <file>` | Start or resume review |
| `/intent-changes propose` | Propose changes |
| `/intent-changes accept <id>` | Accept a change |
| `/intent-changes reject <id>` | Reject a change |
| `/intent-changes status` | View current status |
| `/intent-changes finalize` | Interactive apply |

## Workflow

```
/intent-changes start <file>
        ↓
┌───────────────────┐
│  Check .reviews/  │
│  Resume if exists  │
│  Create if not     │
└─────────┬─────────┘
          ↓
/intent-changes propose
        ↓
┌───────────────────────────────────────┐
│  Read source document                 │
│  Discuss changes with user            │
│  Generate Change Proposal (C001...)   │
│  Write to .reviews/{name}.review.md   │
└─────────┬─────────────────────────────┘
          ↓
/intent-changes accept/reject
        ↓
┌───────────────────┐
│  Update status     │
│  Record reviewer   │
│  Record timestamp  │
└─────────┬─────────┘
          ↓
/intent-changes finalize
        ↓
┌───────────────────────────────────────┐
│  Show each ACCEPTED change            │
│  Interactive confirm: Apply? [Y/n/view]│
│  Apply to source document             │
│  Generate change summary              │
└───────────────────────────────────────┘
```

## Execution Steps

### Command: start

**Input**: File path

**Steps**:

1. Verify source file exists
2. Determine review file path: `.reviews/{basename}.review.md`
3. If review file exists:
   - Read and parse
   - Show current status overview
4. If not exists:
   - Create `.reviews/` directory (if not exists)
   - Create review file with frontmatter
5. Set current session source and review paths

**Output**:
```
Review session started.

Source: intent/specs/kind-system-spec.md
Review: .reviews/kind-system-spec.review.md

Status: 3 PENDING, 2 ACCEPTED, 1 REJECTED

Commands:
  /intent-changes propose  - Propose new changes
  /intent-changes status   - View details
  /intent-changes finalize - Apply changes
```

### Command: propose

**Prerequisite**: Must have run start

**Steps**:

1. Read source document content
2. Read current review file, get existing proposals
3. Calculate next ID (if already has C001-C005, next is C006)
4. Discuss with user:
   - Show source document structure
   - Use AskUserQuestion to ask about change type and location
   - Collect change content
5. Generate Change Proposal block
6. Append to review file

**Interactive flow**:

```
AskUserQuestion:
- question: "Which section do you want to propose changes for?"
- header: "Change Location"
- options:
  - "## Kind Definition" - First section
  - "## Actions" - Second section
  - "## Examples" - Third section
  - "Other location" - Specify manually
```

```
AskUserQuestion:
- question: "What type of change?"
- header: "Change Type"
- options:
  - "ADD" - Add new content
  - "MODIFY" - Modify existing content
  - "REPLACE" - Replace entire section
  - "DELETE" - Delete content
```

Then collect specific content and generate the proposal.

### Command: accept / reject

**Input**: Proposal ID, optional comment/reason

**Syntax**:
```
/intent-changes accept C001
/intent-changes accept C001 --comment "LGTM"
/intent-changes reject C002 --reason "Disagree with this approach"
```

**Steps**:

1. Read review file
2. Find proposal with matching ID
3. Verify current status is PENDING
4. Update status to ACCEPTED or REJECTED
5. Add Decision record:
   - reviewer: from git config user.name or $USER
   - timestamp: current date
   - comment: user-provided comment
6. Write back to review file

**Decision format**:
```markdown
**Decision:**
- ✓ @robmao (2026-01-21): "LGTM"
```

Or when rejected:
```markdown
**Decision:**
- ✗ @robmao (2026-01-21): "Disagree with this approach"
```

### Command: status

**Detailed output**:

```
Review: kind-system-spec.md
Source: intent/specs/kind-system-spec.md
Reviewers: @robmao, @claude

─────────────────────────────────

PENDING (2):
  C002 [MODIFY] Revise Kind definition wording
  C004 [ADD] Add performance chapter

ACCEPTED (3):
  C001 [ADD] Add Action classification description
       ✓ @robmao (2026-01-21)
  C003 [MODIFY] Adjust example code
       ✓ @claude (2026-01-21)
  C005 [DELETE] Remove outdated chapter
       ✓ @robmao (2026-01-21)

REJECTED (1):
  C006 [REPLACE] Rewrite entire document
       ✗ @robmao (2026-01-21): "Too many changes"

─────────────────────────────────

Next: /intent-changes finalize (3 changes ready)
```

### Command: finalize

**Prerequisite**: At least one ACCEPTED proposal

**Steps**:

1. Read review file, filter ACCEPTED proposals
2. Sort by Target position (from end of document to start, to avoid position shifts)
3. Interactive confirmation for each proposal:

```
[1/3] C001 [ADD]: Add Action classification description

Target: After "## Actions"

Content to add:
┌────────────────────────────────────────────┐
│ Actions are divided into three categories: │
│ - Inline: Synchronous, determines commit   │
│   success                                  │
│ - Deferred: Async, failure doesn't affect  │
│   commit                                   │
│ - Observational: Read-only, can be slow    │
└────────────────────────────────────────────┘

Apply this change? [Y/n/view/quit]
```

User options:
- `Y` (default): Apply and continue
- `n`: Skip this change (keep ACCEPTED status but don't apply)
- `view`: Show full before/after diff
- `quit`: Abort finalize

4. Apply changes to source document
5. Update review file:
   - Mark applied changes as `[APPLIED]`
   - Update frontmatter status to `finalized` (if all processed)
6. Output summary

**Apply summary**:
```
Finalize complete.

Applied: 2
  C001 - Add Action classification description
  C003 - Adjust example code

Skipped: 1
  C005 - Remove outdated chapter (user chose to skip)

Source updated: intent/specs/kind-system-spec.md
Review archived: .reviews/kind-system-spec.review.md
```

## Review File Format

### Frontmatter

```yaml
---
source: intent/specs/kind-system-spec.md
created: 2026-01-21
reviewers:
  - robmao
  - claude
status: active
---
```

Status values:
- `active`: In progress
- `finalized`: Apply completed
- `abandoned`: Abandoned

### Change Proposal Block

```markdown
---

## C001 [ADD] [PENDING]

> Short description

**Target:** After "## Actions"

**After:**
```markdown
New content...
```

---

## C002 [MODIFY] [ACCEPTED]

> Short description

**Target:** Section "## Kind Definition"

**Before:**
```markdown
Original content...
```

**After:**
```markdown
New content...
```

**Reason:** Reason for change

**Decision:**
- ✓ @robmao (2026-01-21): "LGTM"
```

## Getting Reviewer Name

By priority:
1. Command parameter `--reviewer`
2. `git config user.name`
3. Environment variable `$USER`

```bash
# How to get
git config user.name || echo $USER
```

## Boundaries

### What It Does

- ✓ Structured change proposal management
- ✓ Track decision process
- ✓ Support multi-reviewer signatures
- ✓ Interactive apply

### What It Doesn't Do

- ✗ Format validation → intent-validate
- ✗ Section approval → intent-review
- ✗ Design quality judgment → intent-critique
- ✗ Implementation consistency → intent-sync

## Integration with Other Tools

```
intent-interview → Create Intent
        ↓
intent-critique → Challenge design
        ↓
/intent-changes → Manage change proposals  ← This Skill
        ↓
intent-review → Lock sections
        ↓
intent-plan → Start implementation
```

## Examples

### Independent Review

```bash
# Start
/intent-changes start intent/specs/tools-spec.md

# Propose changes
/intent-changes propose

# Decide
/intent-changes accept C001
/intent-changes reject C002 --reason "Not needed"

# Apply
/intent-changes finalize
```

### Collaborative Review

```bash
# A starts and proposes
/intent-changes start spec.md
/intent-changes propose  # C001-C003

# B reviews
/intent-changes accept C001 --comment "Good"
/intent-changes reject C002 --reason "Try a different approach"

# A proposes new alternative
/intent-changes propose  # C004 replaces C002

# B accepts
/intent-changes accept C004

# Final apply
/intent-changes finalize
```
