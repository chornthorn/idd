# Intent Design Standard

> Universal standard for multi-project, multi-repo Intent files
> Version: 1.1

[中文版](zh/intent-standard.md)

## Design Goals

1. **Agent Regional Autonomy** - Each module/chamber has its own intent, agents work autonomously within their scope
2. **Clear Hierarchy** - Project-level, module-level, and architecture-level each have distinct responsibilities
3. **Diagrams First** - ASCII diagrams are more precise than prose
4. **Testability** - Intent maps directly to test assertions

---

## Directory Structure

### Standard Layout

```
project/
├── intent/                      # Project-level intent
│   ├── INTENT.md                # Entry: vision + index
│   ├── decisions.md             # Interview decisions (Phase A output)
│   ├── _archive/                # Inactive intents (shelved, abandoned, superseded)
│   ├── _data/                   # Computed data (generated, never hand-edited)
│   ├── records/                 # Raw input archive (append-only)
│   │   ├── INDEX.md             # Summary index of all records
│   │   ├── interview-2026-02-01.md
│   │   ├── review-chatgpt-2026-02-03.md
│   │   └── critique-2026-02-05.md
│   ├── architecture/            # Architecture constraints
│   │   ├── DEPENDENCIES.md      # Module dependency graph
│   │   └── BOUNDARIES.md        # Boundary rules
│   └── specs/                   # Structure specifications
│       └── *.md
│
├── src/
│   ├── core/                    # Core module
│   │   └── intent/
│   │       ├── INTENT.md        # Core module intent
│   │       ├── decisions.md     # Interview decisions
│   │       └── records/         # Raw input archive
│   │           └── INDEX.md
│   │
│   ├── platforms/               # Multi-platform
│   │   ├── web/
│   │   │   └── intent/
│   │   │       └── INTENT.md
│   │   └── cli/
│   │       └── intent/
│   │           └── INTENT.md
│   │
│   └── chambers/                # Independent chambers
│       ├── terminal/
│       │   └── intent/
│       │       └── INTENT.md
│       └── portal/
│           └── intent/
│               └── INTENT.md
│
└── docs/
    └── engineering/
        └── intent-standard.md   # This document
```

---

## Hierarchy Responsibilities

```
┌─────────────────────────────────────────────────────────────┐
│  Project-level intent/                                       │
│  - Vision and goals                                          │
│  - Module index                                              │
│  - Architecture constraints (dependencies, boundaries)       │
│  - Global structure specifications                           │
├─────────────────────────────────────────────────────────────┤
│  Module-level intent/                                        │
│  - Module responsibilities                                   │
│  - API signatures                                            │
│  - Internal structure                                        │
│  - Module-specific specifications                            │
│                                                              │
│  ⚠️ Agents work autonomously here, no boundary crossing      │
└─────────────────────────────────────────────────────────────┘
```

### Project-level vs Module-level

| Content | Project-level | Module-level |
|---------|---------------|--------------|
| Vision & Goals | ✓ | - |
| Module Index | ✓ | - |
| Dependencies | ✓ | - |
| Boundary Rules | ✓ | - |
| Module Responsibilities | - | ✓ |
| API Signatures | - | ✓ |
| Internal Structure | - | ✓ |
| Test Assertions | - | ✓ |

---

## Frontmatter

All intent files (INTENT.md, decisions.md, etc.) **may** have YAML frontmatter for machine-readable metadata:

```markdown
---
status: active
---
# <ModuleName> Intent

> Anchor: ...
```

**Rules:**
- Frontmatter is optional — omitted means `status: active`
- Keep it minimal. Only add fields that tools consume. Prose belongs in the document body.
- Frontmatter is excluded from Size Budget line counting

**Allowed fields:**

| Field | Values | Description |
|-------|--------|-------------|
| `status` | `active`, `implemented`, `shelved`, `abandoned`, `superseded` | Intent lifecycle state |
| `superseded_by` | intent ID | Only when `status: superseded` |
| `split_into` | intent ID list | Only when archived after split |

**Examples:**

```yaml
---
status: implemented
---
```

```yaml
---
status: superseded
superseded_by: auth/v2
---
```

---

## File Formats

### INTENT.md (Entry Point)

```markdown
# <ModuleName> Intent

> Anchor: One sentence declaring why this module exists    ← REQUIRED

## Responsibilities

- What it does
- What it doesn't do

## Structure

\```
Directory/data structure diagram
\```

## API

### functionName(params)

**Parameters:** ...
**Returns:** ...
**Constraints:** ...

## Examples

Input → Output
```

### DEPENDENCIES.md (Dependency Graph)

```markdown
# Module Dependencies

## Dependency Graph

\```
    API Layer
        │
   ┌────┴────┐
   ▼         ▼
 Deploy   Router
   │         │
   └────┬────┘
        ▼
    Chamber
        │
        ▼
       FS
\```

## Dependency Matrix

|          | Chamber | Deploy | Router | FS |
|----------|---------|--------|--------|-----|
| API      | ✓       | ✓      | ✓      | ✗   |
| Deploy   | ✓       | -      | ✗      | ✗   |
| Router   | ✓       | ✗      | -      | ✗   |
| Chamber  | -       | ✗      | ✗      | ✓   |

✓ Allowed  ✗ Forbidden  - Self
```

### BOUNDARIES.md (Boundary Rules)

```markdown
# Boundary Rules

## Principles

- Dependencies flow one way: upper → lower
- No skipping layers
- Communicate via API, no direct internal access

## Forbidden Patterns

| File | Forbidden | Reason |
|------|-----------|--------|
| api.js | `join(appsDir, ...)` | Must use Chamber API |
| deploy.js | `import.*route-engine` | No cross-module imports |

## Verification

\```bash
# Detect violations
grep -r "join.*appsDir" src/platforms/
\```
```

---

## Anchor

Every INTENT.md **must** have an anchor statement — one or two sentences immediately after the title, in blockquote format:

```markdown
# Session Protocol Intent

> Anchor: Enable type-safe binary communication between AFS nodes.
> Assumes: kernel/proc, kernel/afs-scoping
```

**Rules:**
- One or two sentences max, no paragraphs
- Declares the module's reason to exist
- Every section in the file must trace back to this anchor
- If a section cannot justify its relevance to the anchor, it should be removed

### Assumes Tag

The `Assumes:` line is **optional** — only add it when the intent depends on other intents.

```markdown
> Assumes: kernel/proc, kernel/afs-scoping
```

**Rules:**
- Comma-separated list of intent IDs (relative paths from `intent/`)
- Only declare direct dependencies, not transitive ones
- Used by `intent-audit` to build the dependency graph and detect orphans/stale refs
- See [intent-data-format.md](intent-data-format.md) for the generated graph schema

---

## Records

Every intent directory **should** have a `records/` subdirectory for raw input archiving:

```
intent/module-name/
├── INTENT.md              # Distilled spec (budget-controlled)
├── decisions.md           # Interview Phase A output
└── records/               # All raw inputs, append-only
    ├── INDEX.md            # Summary index
    ├── interview-2026-02-01.md
    ├── review-chatgpt-2026-02-03.md
    ├── review-gemini-2026-02-04.md
    ├── critique-2026-02-05.md
    └── import-2026-02-06.md
```

**Principles:**
- `records/` is append-only — never delete, never edit existing records
- No budget constraint on records — raw materials can be any length
- File naming: `{type}-{source?}-{YYYY-MM-DD}.md` (source optional for internal ops)
- `INDEX.md` maintains a one-line summary per record for quick scanning

**INDEX.md format:**

```markdown
# Records Index

| Date | Type | Source | Summary |
|------|------|--------|---------|
| 2026-02-01 | interview | — | Initial interview, 12 decisions on data model and API |
| 2026-02-03 | review | chatgpt | Architecture review, recommended splitting storage layer |
| 2026-02-05 | critique | — | Convergence check, removed 3 sections (-120 lines) |
```

**Usage:**
- `/intent-interview` saves raw transcript to `records/interview-{date}.md`
- `/intent-critique` saves critique record to `records/critique-{date}.md`
- External AI reviews: user imports to `records/review-{source}-{date}.md`
- Critique reads `records/INDEX.md` first, drills into specific records as needed

---

## Size Budget

Intent files have tiered line limits to prevent scope creep:

| Lines | Status | Action |
|-------|--------|--------|
| ≤ 300 | Healthy | None |
| 300–500 | Warning | Tool prompts "Intent is getting long — split or trim?" |
| > 500 | Block | Must run `/intent-critique` before adding any new content |

**Counting rules:**
- Count all lines including blank lines and code fences
- Frontmatter (if any) is excluded from the count

**Data basis:** 19 stable intents average 190 lines. Healthy intents with implementation-driven growth stay in 300–700. Intents exceeding 500 almost always have accretion problems.

---

## Anti-Accretion Rules

Intent files tend to grow over time. These rules prevent unbounded accretion:

1. **Anchor guard** — Every section must serve the anchor statement. Content that drifts from the anchor belongs in a different Intent or nowhere.
2. **Size budget** — ≤ 300 healthy, 300–500 warning, > 500 blocked. Data-driven thresholds.
3. **Interview only asks** — `/intent-interview` outputs decisions, not new sections. The Intent file is composed separately under budget constraints.
4. **Critique must net-reduce** — After `/intent-critique`, total line count must be ≤ before. Only delete, merge, split, or simplify — never add new concepts.
5. **Convergence checkpoint** — After 3+ modifications, `/intent-critique` auto-runs convergence checks: anchor trace, implementation match, constraint-vs-analysis classification, dedup with sub-intents.

---

## Agent Work Mode

### Regional Autonomy

```
┌──────────────────────────────────────────────────────────────┐
│                     Project-level Agent                       │
│  - Read intent/INTENT.md to understand the whole             │
│  - Assign tasks to specific modules                          │
│  - Does not directly modify module internals                 │
└──────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│  Core Agent      │ │  Terminal Agent  │ │  CLI Agent       │
│                  │ │                  │ │                  │
│  Reads:          │ │  Reads:          │ │  Reads:          │
│  core/intent/    │ │  terminal/intent/│ │  cli/intent/     │
│                  │ │                  │ │                  │
│  Scope:          │ │  Scope:          │ │  Scope:          │
│  core/src/       │ │  terminal/src/   │ │  cli/src/        │
│  core/tests/     │ │  terminal/tests/ │ │  cli/tests/      │
└──────────────────┘ └──────────────────┘ └──────────────────┘
```

### Key Rules

1. **Agents work only in their region** - No crossing boundaries
2. **Agents decompose tasks autonomously** - No external task files needed
3. **Agents follow local intent** - Read intent first, then plan
4. **Cross-module needs escalate** - Generate requests when needing to modify other modules

---

## Minimalism Principle

### Must Have

- `intent/INTENT.md` - Project entry
- `<module>/intent/INTENT.md` - Each independent module

### Add As Needed

- `intent/architecture/` - When multi-module dependencies exist
- `intent/specs/` - When complex structure specifications needed
- `<module>/intent/*.md` - When module has multiple design docs

### Not Needed

- Task files - Agents decompose autonomously
- User Story files - Describe behavior directly
- Classification by requirement type - Organize by module

---

## Evolution Path

```
Stage 1: Minimal
├── intent/INTENT.md              # Project entry
└── module/intent/INTENT.md       # Each module

Stage 2: Add Architecture Constraints
├── intent/
│   ├── INTENT.md
│   └── architecture/
│       ├── DEPENDENCIES.md
│       └── BOUNDARIES.md

Stage 3: Add Structure Specifications
├── intent/
│   ├── INTENT.md
│   ├── architecture/
│   └── specs/
│       └── directory-structure.md
```

---

## Lifecycle & Archive

Intents that are no longer active move to `intent/_archive/`. Directory structure is the lifecycle mechanism — `git mv` is the state transition, commit message is the reason.

```
intent/
├── kernel/proc/INTENT.md        # active
├── kernel/afs/INTENT.md         # active
└── _archive/
    ├── kernel/echo/INTENT.md    # abandoned
    └── auth/legacy/INTENT.md    # superseded by auth/v2
```

**Lifecycle transitions:**

| Event | Action |
|-------|--------|
| Shelved / Abandoned | `git mv intent/x → intent/_archive/x`, set frontmatter `status` |
| Split | Original → `_archive/` with `split_into:`, create new intents |
| Merged | Absorbed intent → `_archive/`, target stays |
| Superseded | Old → `_archive/` with `superseded_by:`, new in `intent/` |
| Revived | `git mv intent/_archive/x → intent/x`, set `status: active` |
| Implemented | Stays in `intent/` — it's the spec for living code, set `status: implemented` |

**Rules:**
- `_archive/` preserves the original directory structure for traceability
- Archived intents retain their `records/` for history
- `intent-audit` ignores `_archive/` — no health checks, no orphan counting
- Git history is the event log — no separate lifecycle file needed

---

## TASK.yaml

Each intent can have a `TASK.yaml` for execution tracking. Generated by `/intent-plan`.

### Base Fields

```yaml
status: ready           # ready | in_progress | review | done
owner: null             # assigned agent/team
assignee: null          # current executor
phase: 0/3              # current/total phases
updated: "2026-02-06T10:00:00Z"
heartbeat: null         # last activity timestamp
```

### Task Type Extension

Tasks default to one-shot execution. For recurring or event-driven work, declare the type:

```yaml
type: recurring              # one-shot | scheduled | recurring | event
schedule: "2026-02-14T09:00" # scheduled: specific execution time
recurrence: "weekly:mon"     # recurring: cadence pattern
trigger: "version-released"  # event: trigger condition
last_run: null               # last execution timestamp
run_count: 0                 # total executions
```

**Type definitions:**

| Type | Fields Used | Description |
|------|-------------|-------------|
| `one-shot` | (none) | Default. Execute once, mark done. |
| `scheduled` | `schedule` | Execute at a specific time. |
| `recurring` | `recurrence` | Execute on a cadence. |
| `event` | `trigger` | Execute when condition is met. |

**Recurrence patterns:** `daily`, `weekly:mon`, `weekly:mon,thu`, `monthly:1` (day of month), `sprint` (per sprint boundary).

**Note:** IDD defines the format. TeamSwarm interprets and schedules execution.

---

## Related Documents

- [intent-approval.md](intent-approval.md) - Section-level approval mechanism
- [intent-data-format.md](intent-data-format.md) - Machine-readable data schemas

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.1 | 2026-02-06 | Add `Assumes:` tag, TASK.yaml type extension, frontmatter, lifecycle & archive |
| 1.0 | 2026-01-19 | Initial version |
