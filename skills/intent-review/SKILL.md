---
name: intent-review
description: Interactive Intent approval. Review sections and mark status (locked/reviewed/draft). Use /intent-review <path> to review a specific file, or /intent-review to review Intent in current directory.
---

# Intent Review

Interactive section-level Intent approval tool.

## Core Concepts

Intent documents are approved at section granularity, with three statuses:

| Status | Marker | Meaning | Agent Behavior |
|--------|--------|---------|----------------|
| **LOCKED** | `::: locked` | Core architecture, modifications require explicit human consent | Pause, request confirmation |
| **REVIEWED** | `::: reviewed` | Reviewed, can be modified but notification required | Allow, notify afterwards |
| **DRAFT** | `::: draft` | Draft, can be freely iterated | Free to modify |

## Workflow

```
/intent-review [path]
        ↓
┌───────────────────┐
│  Parse Intent file │
│  Identify sections │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│  Show status       │
│  overview          │
│  N locked          │
│  M reviewed        │
│  K draft/unmarked  │
└─────────┬─────────┘
          ↓
┌───────────────────────────────────────┐
│  Ask about each unmarked section      │
│                                       │
│  AskUserQuestion:                     │
│  "Section: [title]"                   │
│  Options:                             │
│  - Lock (core architecture)           │
│  - Review (confirm acceptance)        │
│  - Skip (keep as draft)               │
└─────────┬─────────────────────────────┘
          ↓
┌───────────────────┐
│  Update file       │
│  Add markers and   │
│  attributes        │
└───────────────────┘
```

## Usage

### Review a specific file

```
/intent-review src/core/intent/INTENT.md
```

### Review current module

```
cd src/chambers/terminal
/intent-review
```

Automatically finds `intent/INTENT.md`

### Batch review

```
/intent-review --all
```

Scans all intent files in the project

## Execution Steps

### 1. Locate Intent File

```javascript
// Priority:
// 1. User-specified path
// 2. intent/INTENT.md in current directory
// 3. INTENT.md in current directory
```

### 2. Parse Sections

Identify two types of sections:

**Marked sections**:
```markdown
::: locked
## Module Boundaries
...
:::
```

**Unmarked sections** (plain markdown headings):
```markdown
## API Design
...
```

### 3. Show Overview

```
Intent Review: src/core/intent/INTENT.md

Status Overview:
├── 🔒 LOCKED: 2 sections
│   ├── Module Boundary Rules
│   └── Data Structures
├── ✓ REVIEWED: 3 sections
│   ├── API Signatures (by robert, 2026-01-19)
│   ├── Config Format
│   └── Error Handling
└── 📝 UNMARKED: 4 sections
    ├── Implementation Suggestions
    ├── Performance Considerations
    ├── Example Code
    └── Change Log

Start reviewing unmarked sections?
```

### 4. Review Each Section

Use AskUserQuestion for each unmarked section:

```
AskUserQuestion:
- question: "Section: API Signatures\n\nContent preview: Defines three functions create(), delete(), list()..."
- header: "Approval Status"
- options:
  - label: "Lock (core architecture)"
    description: "Lock this section, modifications require explicit human consent"
  - label: "Review (confirm acceptance)"
    description: "Mark as reviewed, Agent can modify but must notify"
  - label: "Skip (keep as draft)"
    description: "Do not review yet, keep as draft"
```

### 5. Update File

Based on user selection, add markers to the markdown:

**Lock selection**:
```markdown
::: locked {reason="core architecture"}
## API Signatures
...
:::
```

**Review selection**:
```markdown
::: reviewed {by=<username> date=<today>}
## API Signatures
...
:::
```

**Skip selection**:
No modification, or add:
```markdown
::: draft
## Implementation Suggestions
...
:::
```

## Special Scenarios

### Detecting Modifications to LOCKED Sections

When the file has uncommitted changes:

```
⚠️ LOCKED section modification detected:
- Section: Module Boundary Rules
- Changes: [diff preview]

This modification requires explicit human consent. Accept?
```

### View Approval History

```
/intent-review --history src/core/intent/INTENT.md
```

Output:
```
Section: API Signatures
├── 2026-01-19 robert: reviewed
├── 2026-01-15 robert: draft → reviewed
└── 2026-01-10 created

Section: Module Boundaries
├── 2026-01-18 robert: locked (reason: core architecture)
└── 2026-01-12 created
```

## Integration with Other Tools

```
intent-interview → Create Intent (all sections default to draft)
        ↓
/intent-review → Approve key sections
        ↓
aine-dev-flow → Develop implementation (respects locked/reviewed rules)
        ↓
intent-sync → Check consistency
```

## Configuration

Configure the default reviewer in `~/.claude/settings.json`:

```json
{
  "idd": {
    "reviewer": "robert",
    "autoLockPatterns": ["Module Boundaries", "Data Structures", "Security Constraints"]
  }
}
```
