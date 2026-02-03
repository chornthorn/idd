# /intent-plan

> Generate TaskSwarm-compatible execution plan with strict TDD

## Usage

```bash
/intent-plan
/intent-plan path/to/INTENT.md
```

## Purpose

Transforms an approved Intent into a structured, executable development plan. The output is **TaskSwarm-compatible** and can be directly executed by `/swarm run`.

## Pre-Plan Gates (Mandatory)

**Before generating any plan, ALL gates must pass:**

```
/intent-plan
    ↓
┌─────────────────────────────────────┐
│ Gate 1: Interview Check             │
│   - INTENT.md exists and complete   │
│   ✗ → "Run /intent-interview first" │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Gate 2: Critique Check              │
│   - Has been critiqued              │
│   - Re-critique if modified after   │
│   ✗ → "Run /intent-critique"        │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Gate 3: Dependency Check            │
│   - Prerequisites satisfied         │
│   - Blocking intents are done       │
│   ✗ → "Complete dependencies first" │
└─────────────────────────────────────┘
    ↓
✓ All gates passed → Generate plan.md
```

### Gate Failure Examples

```
❌ Gate 1 Failed: Interview Incomplete
   INTENT.md missing Structure section
   Action: Run /intent-interview

❌ Gate 2 Failed: Critique Required
   INTENT.md modified after last critique (2026-01-15)
   Action: Run /intent-critique

❌ Gate 3 Failed: Dependencies Not Ready
   Blocking: intent/auth-module/ (status: in_progress)
   Action: Complete blocking tasks first
```

## Output Files

### 1. plan.md

Execution plan containing:
- Phase structure (0-indexed)
- 6-category test matrix per phase
- E2E Gate verification scripts
- Checkbox tracking (`- [ ]`)

### 2. TASK.yaml

TaskSwarm status file:
```yaml
status: ready
owner: null
assignee: null
phase: 0/N
updated: 2024-01-21T10:00:00Z
heartbeat: null
```

## Plan Structure

```
## Phase 0: [Phase Name]
├── ### Description
├── ### Tests
│   ├── #### Happy Path
│   │   └── - [ ] test case 1
│   ├── #### Bad Path
│   │   └── - [ ] error case 1
│   ├── #### Edge Cases
│   │   └── - [ ] boundary case 1
│   ├── #### Security
│   │   └── - [ ] vulnerability test 1
│   ├── #### Data Leak
│   │   └── - [ ] leak prevention test 1
│   └── #### Data Damage
│       └── - [ ] integrity test 1
├── ### E2E Gate
│   └── CLI/Script verification
└── ### Acceptance Criteria
    └── - [ ] criterion 1

## Phase 1: [Phase Name]
└── ...
```

## Test Categories (All 6 Required)

Every phase MUST include tests from all categories:

| Category | Purpose | Examples |
|----------|---------|----------|
| **Happy Path** | Normal usage | Valid inputs, correct sequences |
| **Bad Path** | Error conditions | Wrong types, missing fields |
| **Edge Cases** | Boundary conditions | Empty inputs, max values |
| **Security** | Vulnerability prevention | Injection, auth bypass |
| **Data Leak** | Information exposure | Sensitive data in logs |
| **Data Damage** | Data integrity | Partial writes, corruption |

**Bad cases must be detailed and comprehensive.** More failure tests than success tests.

## Example Session

```
> /intent-plan intent/payment/

Reading Intent...
Analyzing scope and complexity...
Generating phases...

## Execution Plan Generated

### Overview
- Total Phases: 2
- Total Tests: 28
- E2E Gates: 2

### Phase 0: Payment Validation
- 15 test cases across 6 categories
- E2E: CLI payment validation script

### Phase 1: Payment Processing
- 13 test cases across 6 categories
- E2E: Full transaction verification

Files created:
- intent/payment/plan.md
- intent/payment/TASK.yaml (status: ready)

Next steps:
1. Review plan.md
2. Run /swarm run payment (with TaskSwarm)
   Or /intent-build-now (without TaskSwarm)
```

## Example Output: plan.md

```markdown
# Execution Plan: payment

## Overview

实现支付验证和处理功能。

## Prerequisites

- 数据库连接配置完成
- 支付网关 API 密钥已配置

## Phase 0: Payment Validation

### Description

实现支付金额和参数验证。

### Tests

#### Happy Path
- [ ] validates correct amount format
- [ ] accepts valid currency codes
- [ ] accepts valid card token

#### Bad Path
- [ ] rejects negative amount
- [ ] rejects zero amount
- [ ] rejects invalid currency
- [ ] rejects expired card token

#### Edge Cases
- [ ] handles maximum amount (system limit)
- [ ] handles minimum amount (1 cent)

#### Security
- [ ] rejects SQL injection in metadata
- [ ] validates input length limits

#### Data Leak
- [ ] card number not in error messages
- [ ] card number not in logs

#### Data Damage
- [ ] validation failure doesn't create partial record

### E2E Gate

```bash
# Verify validation works
curl -X POST http://localhost:3000/api/payments/validate \
  -H "Content-Type: application/json" \
  -d '{"amount": 1000, "currency": "USD"}' | jq .

# Verify rejection
curl -X POST http://localhost:3000/api/payments/validate \
  -d '{"amount": -100}' | jq .status  # Should be "error"
```

### Acceptance Criteria

- [ ] 所有 6 类测试通过
- [ ] E2E Gate 验证通过
- [ ] 代码已提交

---

## Phase 1: Payment Processing

[...]
```

## Integration with TaskSwarm

```
/intent-plan
    ↓
Generates plan.md + TASK.yaml
    ↓
/swarm run
    ↓
TaskSwarm executes TDD phases
    ↓
Checkboxes updated: - [ ] → - [x]
    ↓
Git commits after each phase
```

## vs /intent-build-now

| Aspect | /intent-plan | /intent-build-now |
|--------|--------------|-------------------|
| Output | plan.md + TASK.yaml | Validates then executes |
| Use case | Generate plan first | Start execution now |
| Review | Plan review before execution | Immediate start |

## Best Practices

1. **0-indexed phases** - Phase 0, Phase 1, Phase 2...
2. **Checkbox format** - All tests use `- [ ]`
3. **Specific test cases** - "rejects negative amount" not "test errors"
4. **E2E Gate per phase** - Must be runnable via CLI
5. **Bad Path > Happy Path** - More failure tests than success tests

## Related Commands

- [/intent-build-now](intent-build-now.md) - Validate and execute
- [/intent-review](intent-review.md) - Approve before planning
- [/intent-sync](intent-sync.md) - Sync after execution

## See Also

- [IDD Workflow](../workflow.md) - Planning in context
- [TaskSwarm](https://github.com/ArcBlock/teamswarm) - Autonomous execution
