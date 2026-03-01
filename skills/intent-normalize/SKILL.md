---
name: intent-normalize
description: Scan and normalize existing intent/planning files to IDD standard. Auto-fixes mechanical issues (frontmatter, directory structure), tags content issues for pickup. Use /intent-normalize to scan current project, or /intent-normalize <path> for specific directory.
---

# Intent Normalize

Scan existing projects and normalize intent and planning files to the IDD standard. Auto-fixes mechanical issues directly, tags content issues for later pickup by TeamSwarm.

## Design Principles

1. **Format first** — All files get frontmatter first, so the system can identify them
2. **Auto-fix when possible** — Mechanical issues are fixed directly (frontmatter, directory structure, naming)
3. **Tag when not possible** — Content issues are tagged with status for later processing by TeamSwarm or humans
4. **Idempotent** — Multiple runs produce the same result; already normalized files are not modified

## Workflow

```
/intent-normalize [path]
        ↓
┌───────────────────────────────────┐
│  1. Scan intent/ and planning/    │
│     Discover all .md and .yaml    │
│     files                         │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  2. Analyze compliance status     │
│     of each file                  │
│     - Has frontmatter?            │
│     - Has Anchor?                 │
│     - Directory structure          │
│       complete?                   │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  3. Auto-fix mechanical issues    │
│     - Add frontmatter             │
│     - Create records/ + INDEX.md  │
│     - Create _archive/, _data/    │
│     - Normalize file naming       │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  4. Tag content issues            │
│     - Missing Anchor →            │
│       status: draft               │
│     - Over budget →               │
│       tag needs_critique          │
│     - Cannot classify →           │
│       status: draft               │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  5. Output normalization report   │
└───────────────────────────────────┘
```

## Checks and Fix Strategies

### Auto-fix (Mechanical, No Judgment Required)

| Check | Condition | Fix Action |
|-------|-----------|------------|
| Intent frontmatter | INTENT.md has no frontmatter | Add `status: active` (or infer from content) |
| Planning frontmatter | .md files under planning/ have no frontmatter | Add `type:` (inferred from content: idea/research/feedback/analysis) |
| Anchor format | Anchor exists but format is incorrect (not a blockquote) | Fix to `> Anchor: ...` format |
| records/ directory | No records/ under intent directory | Create `records/` + empty `INDEX.md` |
| _archive/ directory | No _archive/ under intent/ | Create `intent/_archive/` |
| _data/ directory | No _data/ under intent/ | Create `intent/_data/` |
| TASK.yaml format | TASK.yaml missing type field | Add `type: one-shot` (default) |

### Tag for Pickup (Requires Content Judgment)

| Check | Condition | Tag Action |
|-------|-----------|------------|
| Missing Anchor | INTENT.md has no Anchor line | Add `needs: [anchor]` to frontmatter |
| Over budget | Line count > 500 | Add `needs: [critique]` to frontmatter |
| Budget warning | 300 < line count ≤ 500 | Add `needs: [review]` to frontmatter |
| No Assumes | Content references other intents but has no Assumes tag | Add `needs: [assumes]` to frontmatter |
| Unclassifiable | Planning file type cannot be auto-determined | `type: idea` (safest default) |

### `needs` Field

`needs` is a pickup signal in frontmatter, indicating the file needs further processing:

```yaml
---
status: active
needs: [anchor, critique]
---
```

TeamSwarm scans the `needs` field to automatically schedule follow-up skills:
- `needs: [anchor]` → Schedule `/intent-interview` to fill in
- `needs: [critique]` → Schedule `/intent-critique` to simplify
- `needs: [review]` → Schedule `/intent-review`
- `needs: [assumes]` → Schedule manual dependency declaration

After processing, the corresponding `needs` item is removed. When `needs` is empty, the entire field can be deleted.

## Planning Type Inference

Automatically infer `type:` from file content:

| Keywords/Patterns | Inferred Type |
|-------------------|---------------|
| idea, concept, proposal, brainstorm, what if | `idea` |
| research, analysis, comparison, benchmark, study | `research` |
| feedback, user, complaint, request, survey | `feedback` |
| code analysis, refactor, migration, legacy, improvement | `analysis` |
| Cannot determine | `idea` (safest default) |

## Execution Details

### 1. Scan Scope

```
Default scans under the current project root:
- intent/           → All INTENT.md files (recursive, skips _archive/ and _data/)
- planning/         → All .md files
- **/intent/        → Module-level intent directories
```

### 2. Frontmatter Injection

For files without frontmatter, insert at the file header:

**Intent files:**
```yaml
---
status: active
---
```

If the file appears to be already implemented (has corresponding code directory and tests), infer as:
```yaml
---
status: implemented
---
```

**Planning files:**
```yaml
---
type: research
---
```

### 3. Status Inference Heuristics

| Signal | Inferred Status |
|--------|-----------------|
| Has TASK.yaml with status: done | `implemented` |
| Has corresponding src/ directory and file not recently modified | `implemented` |
| File content is sparse (< 50 lines) or clearly incomplete | `active` (+ `needs: [anchor]`) |
| Default | `active` |

### 4. Idempotency Guarantee

- Files with correct frontmatter are not modified
- Existing records/ directories are not recreated
- `needs` field only adds, does not overwrite (merges with existing)
- Each run produces the same result

## Output

### Normalization Report

```markdown
# Intent Normalize Report

> Project: my-project
> Scan time: 2026-02-06

## Statistics

| Type | Scanned | Already Normalized | Auto-fixed | Needs Pickup |
|------|---------|-------------------|------------|--------------|
| Intent files | 15 | 8 | 5 | 2 |
| Planning files | 7 | 2 | 4 | 1 |
| Directory structure | — | — | 3 | — |

## Auto-fixed (Completed)

1. ✅ `intent/kernel/proc/INTENT.md` — Added frontmatter `status: active`
2. ✅ `intent/kernel/afs/INTENT.md` — Added frontmatter `status: implemented`
3. ✅ `planning/v2-ideas.md` — Added frontmatter `type: idea`
4. ✅ `planning/legacy-analysis.md` — Added frontmatter `type: analysis`
5. ✅ `intent/kernel/proc/records/` — Created directory + INDEX.md
6. ✅ `intent/_archive/` — Created directory
7. ✅ `intent/_data/` — Created directory

## Needs Pickup (Tagged)

| File | needs | Suggestion |
|------|-------|------------|
| `intent/ash/INTENT.md` | `[anchor, critique]` | Missing Anchor, 612 lines over budget |
| `intent/surfaces/INTENT.md` | `[anchor]` | Missing Anchor |
| `planning/random-notes.md` | — | Cannot classify, default type: idea |
```

## Usage

### Basic Usage

```
/intent-normalize
```

Scan and normalize the current project.

### Specify Path

```
/intent-normalize intent/kernel/
```

Only normalize a specific directory.

### Dry Run

```
/intent-normalize --dry-run
```

Report only, no modifications.

## Integration with Other Commands

```
/intent-normalize       # Normalize format first (this command)
    ↓
TeamSwarm pickup        # Auto-schedule follow-up skills from needs
    ↓
/intent-interview       # Handle needs: [anchor]
/intent-critique        # Handle needs: [critique]
/intent-review          # Handle needs: [review]
    ↓
/intent-check           # Final compliance check
```
