---
name: intent-build-now
description: Start implementation from Intent. Validates Intent completeness, then either delegates to TaskSwarm (if available) or executes TDD phases directly. Use when you have an Intent ready and want to start building.
---

# Intent Build Now

Start building from Intent. Validates completeness, then chooses execution path.

## Workflow

```
/intent-build-now [path]
       ↓
Locate Intent + plan.md
       ↓
Validate completeness ──→ Incomplete? ──→ Show gaps, suggest /intent-plan
       ↓
Check for TaskSwarm
       │
       ├── Has TaskSwarm + TASK.yaml ──→ Delegate to /swarm run
       │
       └── No TaskSwarm ──→ Execute TDD phases directly
```

## Step 1: Locate Files

Search for Intent files in order:
1. User-specified path
2. `intent/{name}/` directories
3. Current directory

Required files:
- `INTENT.md` - The intent document
- `plan.md` - Execution plan with test matrix

Optional:
- `TASK.yaml` - TaskSwarm status file

If `plan.md` missing:
```
plan.md not found.

Run /intent-plan first to generate the execution plan.
```

## Step 2: Validate Intent Completeness

### Required Sections in INTENT.md

| Section | Purpose | How to Fix |
|---------|---------|------------|
| **Responsibilities** | What it does / doesn't do | Use `/intent-interview` to clarify scope |
| **Structure** | ASCII diagram of components | Add `## Structure` with ASCII diagram |
| **API** | Function signatures | Define key interfaces in `## API` |

### Required in plan.md

| Check | Criteria |
|-------|----------|
| **Phases** | Has `## Phase 0:` or more phases |
| **Tests** | Each phase has 6 test categories |
| **E2E Gate** | Each phase has `### E2E Gate` |
| **Checkboxes** | Uses `- [ ]` format |

### If Not Ready

```markdown
## Intent Validation: NEEDS WORK

Missing elements:

1. **plan.md missing E2E Gate for Phase 1**
   - Add `### E2E Gate` with verification script

2. **plan.md Phase 0 missing Data Leak tests**
   - Add `#### Data Leak` section with test cases

Run /intent-plan to regenerate, or fix manually.
```

## Step 3: Choose Execution Path

### Path A: TaskSwarm Available

Detect TaskSwarm by checking:
1. `TASK.yaml` exists in intent directory
2. TaskSwarm plugin is installed (check for `/swarm` skill)

If both conditions met:
```markdown
## Intent Validation: PASSED ✓

TaskSwarm detected. Delegating execution...

Your Intent is ready. Run:

    /swarm run {task_name}

Or for continuous execution:

    /swarm run-all

TaskSwarm will:
- Claim the task atomically
- Execute phases with TDD discipline
- Update checkboxes in plan.md
- Commit after each phase
- Push to remote
```

**Do NOT execute phases yourself when TaskSwarm is available.**

### Path B: No TaskSwarm (Direct Execution)

If TaskSwarm not available:
```markdown
## Intent Validation: PASSED ✓

No TaskSwarm detected. Executing directly...

Starting Phase 0: {phase_name}
```

Then execute TDD loop for each phase:

```
For each Phase:
  1. Read ### Tests section
  2. For each unchecked test `- [ ]`:
     a. Write test code (if not exists)
     b. Run test → expect failure (red)
     c. Write implementation
     d. Run test → expect pass (green)
     e. Update plan.md: `- [ ]` → `- [x]`
  3. Run E2E Gate script
  4. Git commit: "feat({scope}): Phase {n} - {phase_name}"
  5. Continue to next Phase
```

## Step 4: Completion

After all phases complete:

```markdown
## Build Complete ✓

All phases executed:
- Phase 0: {name} ✓
- Phase 1: {name} ✓

Next steps:
- Run /intent-sync to update Intent with confirmed details
- Run /intent-check to verify consistency
```

## Integration

```
/intent-interview     # Create Intent from scratch
    ↓
/intent-review        # Section-by-section approval
    ↓
/intent-plan          # Generate plan.md + TASK.yaml
    ↓
/intent-build-now     # THIS SKILL
    ↓
┌───────────────────────────────────────┐
│  Has TaskSwarm?                       │
│    Yes → /swarm run (delegate)        │
│    No  → Execute TDD directly         │
└───────────────────────────────────────┘
    ↓
/intent-sync          # Sync confirmed details back
```

## TDD Execution Standards (Direct Mode)

When executing without TaskSwarm, follow strict TDD:

### Test Categories (All 6 Required)

1. **Happy Path** - Normal expected usage
2. **Bad Path** - Invalid inputs, error conditions
3. **Edge Cases** - Boundary conditions
4. **Security** - Vulnerability prevention
5. **Data Leak** - Information exposure prevention
6. **Data Damage** - Data integrity protection

### TDD Discipline

- **Tests First**: Write ALL tests before implementation
- **Red-Green-Refactor**: Verify test fails, implement, verify pass
- **No Skip**: Every checkbox must be completed
- **Commit Per Phase**: One commit after each phase passes

### E2E Gate Enforcement

Each phase's E2E Gate script MUST pass before proceeding:
```bash
# Run the E2E Gate script from plan.md
# Example: pnpm test -- --grep "Phase 0"

# If fails: Stop and fix
# If passes: Continue to next phase
```

## Tips

1. **Prefer TaskSwarm** - It handles concurrency, heartbeat, and recovery
2. **Don't force incomplete Intents** - Missing plan.md means not ready
3. **Respect checkboxes** - They're the source of truth for progress
4. **Commit frequently** - One commit per phase, not one mega-commit
