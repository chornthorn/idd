# IDD Workflow Guide

> From idea to implementation with Intent-Driven Development

## Overview

IDD (Intent-Driven Development) follows a structured workflow where **Intent is the source of truth**. Unlike traditional development where code drives everything, IDD puts human-readable Intent documents at the center, with AI handling the implementation details.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         IDD Workflow                                 │
│                                                                      │
│   DEFINE          REFINE           BUILD            MAINTAIN         │
│                                                                      │
│  ┌──────┐       ┌──────┐        ┌──────┐         ┌──────┐          │
│  │Assess│──────▶│Create│───────▶│Build │────────▶│ Sync │          │
│  └──────┘       └──────┘        └──────┘         └──────┘          │
│                                                                      │
│  Is IDD right?   Write Intent    Plan + TDD       Keep in sync      │
│  Setup project   Review quality  Strict tests     Validate health   │
│                  Approve sections                                    │
└─────────────────────────────────────────────────────────────────────┘
```

## Phase 1: Define

### Step 1.1: Assess Project Fit

Before adopting IDD, evaluate if it's right for your project.

```bash
/intent-assess
```

**Good fit:**
- New features or modules
- Complex business logic
- Team collaboration needed
- Long-term maintenance expected

**Less suitable:**
- Quick prototypes
- One-off scripts
- Purely mechanical refactoring

See: [/intent-assess command](commands/intent-assess.md)

### Step 1.2: Initialize IDD Structure

Set up the Intent directory structure in your project.

```bash
/intent-init
```

This creates:
```
project/
└── intent/
    └── INTENT.md    # Your first Intent file
```

See: [/intent-init command](commands/intent-init.md)

## Phase 2: Refine

### Step 2.1: Create Intent

Transform your ideas into a structured Intent document through guided interviewing.

```bash
/intent-interview
```

The interview covers:
- **Responsibilities**: What it does and doesn't do
- **Structure**: Component diagrams (ASCII art)
- **API**: Function signatures and contracts
- **Examples**: Input → Output mappings

See: [/intent-interview command](commands/intent-interview.md)

### Step 2.2: Critique Design (Optional)

Review your Intent for over-engineering and YAGNI violations.

```bash
/intent-critique
```

This catches:
- Premature abstractions
- Unnecessary complexity
- Features you won't need

See: [/intent-critique command](commands/intent-critique.md)

### Step 2.3: Review and Approve

Approve Intent sections to lock them as implementation contracts.

```bash
/intent-review
```

Section states:
- **draft**: Work in progress
- **reviewed**: Examined but not finalized
- **locked**: Approved, ready for implementation

See: [/intent-review command](commands/intent-review.md)

### Step 2.4: Propose Changes (When Needed)

For collaborative review with PR-like experience.

```bash
/intent-changes
```

See: [/intent-changes command](commands/intent-changes.md)

## Phase 3: Build

### Step 3.1: Generate Execution Plan

Transform your Intent into a structured, executable plan.

```bash
/intent-plan
```

This command:
1. **Validates prerequisites** (interview, critique, dependencies)
2. **Generates plan.md** with phased execution structure
3. **Creates TASK.yaml** for execution tracking

The plan includes:
- **0-indexed phases** with clear deliverables
- **6 test categories** per phase (Happy/Bad/Edge/Security/Data Leak/Data Damage)
- **E2E Gate** per phase for verification
- **Checkbox tracking** for progress

See: [/intent-plan command](commands/intent-plan.md)

### Step 3.2: Execute with TDD

Once you have a plan, execute it with strict TDD discipline.

```bash
/intent-build-now    # Direct execution
# or
/swarm run           # With TaskSwarm (multi-agent)
```

```
┌─────────────────────────────────────────────────────────────┐
│                    Execution Flow                            │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Read plan.md                                       │    │
│  │  - Phases, tests, acceptance criteria               │    │
│  └─────────────────────┬───────────────────────────────┘    │
│                        ▼                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  For each phase:                                    │    │
│  │  1. Write ALL tests first (6 categories)           │    │
│  │  2. Run tests (expect failure)                     │    │
│  │  3. Implement code                                  │    │
│  │  4. Run tests (expect pass)                        │    │
│  │  5. Run E2E Gate verification                      │    │
│  │  6. Update checkboxes, commit                      │    │
│  └─────────────────────┬───────────────────────────────┘    │
│                        ▼                                     │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  All phases complete                                │    │
│  │  - Final E2E verification                          │    │
│  │  - TASK.yaml status: review                        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**Test Categories (All 6 Required):**

| Category | Purpose |
|----------|---------|
| Happy Path | Normal expected usage |
| Bad Path | Invalid inputs, error conditions |
| Edge Cases | Boundary conditions |
| Security | Vulnerability prevention |
| Data Leak | Information exposure prevention |
| Data Damage | Data integrity protection |

See: [/intent-build-now command](commands/intent-build-now.md)

## Phase 4: Maintain

### Step 4.1: Sync Implementation Details

After implementation, sync confirmed details back to Intent.

```bash
/intent-sync
```

This captures:
- Finalized API signatures
- Actual data structures
- Implementation decisions

See: [/intent-sync command](commands/intent-sync.md)

### Step 4.2: Validate Consistency

Run checks to ensure Intent and implementation stay aligned.

```bash
/intent-check
```

This triggers validation agents automatically.

See: [/intent-check command](commands/intent-check.md)

### Step 4.3: Generate Reports

Create human-readable documentation from Intent files.

```bash
/intent-report
```

See: [/intent-report command](commands/intent-report.md)

### Step 4.4: Share Experience (Optional)

Document your IDD journey for others.

```bash
/intent-story
```

See: [/intent-story command](commands/intent-story.md)

## Quick Reference

### Typical New Feature Flow

```bash
/intent-interview     # Create Intent from your idea
/intent-critique      # Check for over-engineering
/intent-review        # Approve key sections
/intent-build-now     # Validate and start TDD execution
/intent-sync          # Sync back when done
```

### Existing Project Adoption

```bash
/intent-assess        # Evaluate fit
/intent-init          # Set up structure
/intent-interview     # Document existing modules
```

### Health Check

```bash
/intent-check         # Run all validations
```

## Next Steps

- [Command Reference](commands/README.md) - Detailed documentation for each command
- [FAQ](faq.md) - Common questions and answers
- [Intent Standard](intent-standard.md) - File format specification
- [TaskSwarm](https://github.com/ArcBlock/teamswarm) - Autonomous task execution system
