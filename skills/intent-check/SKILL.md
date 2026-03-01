---
name: intent-check
description: Run Intent validation and sync checks. Triggers intent-validate and intent-sync agents. Use /intent-check for full check, or /intent-check --validate/--sync for specific checks.
---

# Intent Check

Triggers Intent validation workflow. A user-friendly entry point for the intent-validate and intent-sync agents.

## Features

1. **Format Validation** (intent-validate) - Check if Intent files comply with IDD standards
2. **Code Sync** (intent-sync) - Check consistency between code implementation and Intent
3. **Combined Report** - Aggregate results from both checks

## Workflow

```
/intent-check [options]
        ↓
┌───────────────────────────────────┐
│  Determine check scope            │
│  - Specified path or current dir  │
│  - Single module or entire project│
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Run intent-validate agent        │
│  → Format compliance report       │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Run intent-sync agent            │
│  → Code consistency report        │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Combined report                  │
│  - Issue list                     │
│  - Fix suggestions                │
│  - Action items                   │
└───────────────────────────────────┘
```

## Usage

### Full Check

```
/intent-check
```

Check the Intent in the current directory, including format validation and code sync.

### Specify Path

```
/intent-check src/core/
```

Check a specific module.

### Format Validation Only

```
/intent-check --validate
```

Only run intent-validate to check Intent file format.

### Code Sync Only

```
/intent-check --sync
```

Only run intent-sync to check code-Intent consistency.

### Full Project Check

```
/intent-check --all
```

Scan and check all Intent files in the project.

### Git Diff Check

```
/intent-check --git-diff origin/main
```

Only check modules that have changes relative to the base branch.

## Output Example

```markdown
# Intent Check Report

> Check time: 2026-01-19 14:30
> Check scope: src/core/

## Overview

| Check Item | Status | Issues |
|------------|--------|--------|
| Format Validation | ⚠️ | 3 |
| Code Sync | ❌ | 5 |

## Format Issues (intent-validate)

### ⚠️ Warnings

1. `src/core/intent/INTENT.md:45`
   - Missing ASCII structure diagram

2. `src/core/intent/INTENT.md:78`
   - API definition missing return value description

### ❌ Errors

1. `src/core/intent/INTENT.md:12`
   - Section marker syntax error: `::: lock` → `::: locked`

## Sync Issues (intent-sync)

### Undocumented Additions

| API | File | Suggestion |
|-----|------|------------|
| `getChamberStats()` | chamber.js:89 | Add to Intent |

### Signature Mismatch

```diff
# deleteChamber
- Intent: deleteChamber(app, name)
+ Code:   deleteChamber(app, name, options)
```

### Boundary Violations

| Rule | Location | Description |
|------|----------|-------------|
| No direct path concatenation | routes/apps.js:45 | Should use chamber.getPath() |

## Recommended Actions

### Fix Immediately (P0)
1. Fix Section marker syntax error
2. Fix boundary violation

### Suggested Fixes (P1)
1. Update Intent: Add `getChamberStats()` API
2. Update Intent: Add options parameter to `deleteChamber`

### Optional Improvements (P2)
1. Add ASCII structure diagram
2. Add API return value descriptions
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All passed |
| 1 | Has warnings |
| 2 | Has errors |

Can be used for CI/CD integration:

```bash
/intent-check || exit 1
```

## Integration with Other Commands

```
/intent-init              # Initialize
    ↓
/intent-interview         # Create Intent
    ↓
/intent-review            # Approve
    ↓
[Development]
    ↓
/intent-check             # ← Check (this command)
    ↓
Fix issues or update Intent
    ↓
/intent-check             # Re-check until passed
```
