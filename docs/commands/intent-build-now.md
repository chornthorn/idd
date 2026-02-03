# /intent-build-now

> Validate Intent completeness and start TDD execution

## Usage

```bash
/intent-build-now
/intent-build-now path/to/intent-directory
```

## Purpose

The bridge between Intent and implementation. This command:

1. **Validates** Intent and plan.md completeness
2. **Reports** any gaps that need fixing
3. **Delegates** to TaskSwarm (if available) or executes directly

## When to Use

- Intent is approved and ready for implementation
- plan.md has been generated via `/intent-plan`
- After `/intent-review` has locked key sections

## Execution Flow

```
┌─────────────────────────────────────────────────────────────┐
│                   /intent-build-now                          │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Step 1: Locate Files                                 │   │
│  │ - INTENT.md                                          │   │
│  │ - plan.md (required)                                 │   │
│  │ - TASK.yaml (optional)                               │   │
│  └────────────────────────┬─────────────────────────────┘   │
│                           ▼                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Step 2: Validate Completeness                        │   │
│  │ - Check required sections                            │   │
│  │ - Verify plan.md has phases + tests                  │   │
│  │ - Assess readiness                                   │   │
│  └────────────────────────┬─────────────────────────────┘   │
│                           │                                  │
│              ┌────────────┴────────────┐                    │
│              ▼                         ▼                    │
│  ┌───────────────────┐     ┌───────────────────────────┐   │
│  │ INCOMPLETE        │     │ READY                     │   │
│  │                   │     │                           │   │
│  │ Show gaps         │     │ Has TaskSwarm?            │   │
│  │ Recommend:        │     │   Yes → /swarm run        │   │
│  │ /intent-plan      │     │   No  → Execute directly  │   │
│  └───────────────────┘     └───────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Validation Checks

### Required Files

| File | Purpose |
|------|---------|
| **INTENT.md** | Intent document |
| **plan.md** | Execution plan with test matrix |

### INTENT.md Checks

| Check | Criteria |
|-------|----------|
| **Responsibilities** | Scope defined |
| **Structure** | ASCII diagram present |
| **API** | Function signatures |

### plan.md Checks

| Check | Criteria |
|-------|----------|
| **Phase Structure** | Has `## Phase 0:` sections |
| **Test Matrix** | 6 categories per phase |
| **E2E Gate** | Each phase has verification |
| **Checkboxes** | Uses `- [ ]` format |

## Example: Ready with TaskSwarm

```
> /intent-build-now

Locating files...
Found: intent/auth/INTENT.md
Found: intent/auth/plan.md
Found: intent/auth/TASK.yaml

Validating...

## Intent Validation: PASSED ✓

✓ INTENT.md complete
✓ plan.md has 3 phases, 47 tests
✓ E2E Gates defined
✓ TASK.yaml present

TaskSwarm detected. Delegating execution...

Your Intent is ready. Run:

    /swarm run auth

Or for continuous execution:

    /swarm run-all
```

## Example: Ready without TaskSwarm

```
> /intent-build-now

Locating files...
Found: intent/payment/INTENT.md
Found: intent/payment/plan.md
No TASK.yaml found

Validating...

## Intent Validation: PASSED ✓

✓ INTENT.md complete
✓ plan.md has 2 phases, 28 tests

No TaskSwarm detected. Executing directly...

Starting Phase 0: Payment Validation

#### Happy Path
- [ ] validates correct amount format
→ Writing test...
→ Running test (expect RED)...
→ Implementing...
→ Running test (expect GREEN)...
- [x] validates correct amount format ✓

[TDD loop continues...]
```

## Example: Not Ready

```
> /intent-build-now

Locating files...
Found: intent/payment/INTENT.md
Missing: plan.md

## Intent Validation: NEEDS WORK

❌ plan.md not found

Run /intent-plan first to generate the execution plan:

    /intent-plan intent/payment/

This will create:
- plan.md (execution plan with test matrix)
- TASK.yaml (for TaskSwarm integration)
```

## TaskSwarm Integration

When TASK.yaml exists and TaskSwarm is available:

| Feature | Benefit |
|---------|---------|
| **Atomic claiming** | No conflicts with other agents |
| **Heartbeat** | Automatic stale task recovery |
| **Multi-agent** | Parallel execution across machines |
| **Progress tracking** | Checkbox updates in plan.md |

## Direct Execution (No TaskSwarm)

When executing directly, follows strict TDD:

1. Read Tests section from plan.md
2. For each unchecked test `- [ ]`:
   - Write test code
   - Run test → expect RED
   - Implement code
   - Run test → expect GREEN
   - Update: `- [ ]` → `- [x]`
3. Run E2E Gate script
4. Git commit
5. Continue to next Phase

## Related Commands

- [/intent-plan](intent-plan.md) - Generate plan.md
- [/intent-review](intent-review.md) - Approve sections before build
- [/intent-sync](intent-sync.md) - Sync back after build

## See Also

- [IDD Workflow](../workflow.md) - Build phase in context
- [TaskSwarm](https://github.com/ArcBlock/teamswarm) - Autonomous execution
