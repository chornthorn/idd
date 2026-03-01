---
name: intent-init
description: Initialize IDD structure in a project. Checks existing state, creates directory structure, and generates templates. Use /intent-init to set up Intent-driven development in current project.
---

# Intent Init

Initialize the IDD (Intent Driven Development) structure for a project.

## Features

1. **Check current state** - Scan the project, identify existing intent files or similar structures
2. **Create directories** - Set up the standard IDD directory structure
3. **Generate templates** - Create entry-point INTENT.md and module-level templates
4. **Configuration suggestions** - Provide configuration recommendations based on project type

## Workflow

```
/intent-init
    ↓
┌───────────────────────────────────┐
│  Phase 1: Scan Current State      │
│  - Search for intent/, specs/,    │
│    docs/                          │
│  - Identify README, DESIGN, and   │
│    other docs                     │
│  - Detect project type            │
│    (monorepo/single module)       │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Phase 2: Present Findings        │
│  - Existing document list         │
│  - Project structure analysis     │
│  - Recommended IDD structure      │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Phase 3: Confirm and Create      │
│  - AskUserQuestion to confirm     │
│    structure                      │
│  - Create directories and         │
│    template files                 │
│  - Optional: Migrate existing docs│
└───────────────────────────────────┘
```

## Standard IDD Directory Structure

### Single Module Project

```
project/
├── intent/
│   └── INTENT.md           # Project Intent (entry point)
├── src/
└── ...
```

### Monorepo / Multi-Module Project

```
project/
├── intent/
│   ├── INTENT.md           # Project overview (entry point)
│   └── architecture/
│       ├── DEPENDENCIES.md # Module dependency graph
│       └── BOUNDARIES.md   # Boundary rules
│
├── src/
│   ├── module-a/
│   │   └── intent/
│   │       └── INTENT.md   # Module Intent
│   └── module-b/
│       └── intent/
│           └── INTENT.md
└── ...
```

## Template Content

### Project-Level INTENT.md

```markdown
# [Project Name] Intent

> One-sentence description of the project goal

Status: draft
Last updated: YYYY-MM-DD

## Vision

[Problem to solve and goals]

## Architecture Overview

[ASCII architecture diagram]

## Module Index

| Module | Responsibility | Intent |
|--------|---------------|--------|
| xxx | ... | [link] |

## Non-Goals

- [What is explicitly not in scope]

## Constraints

- [Technical constraints]
- [Business constraints]
```

### Module-Level INTENT.md

```markdown
# [Module] Intent

> Module responsibility in one sentence

Status: draft
Last updated: YYYY-MM-DD

## Responsibilities

[What the module does]

## Non-Goals

[What the module does not do]

## Data Structures

[Core data structure definitions]

## API

[External interface definitions]

## Examples

[Input → Output examples]
```

## Detection Logic

### Identifying Existing Intent Structure

```javascript
// Check paths
const intentPaths = [
  'intent/INTENT.md',
  'INTENT.md',
  'docs/INTENT.md',
  'specs/',
  'design/',
];

// Check content markers
const intentMarkers = [
  '## Responsibilities',
  '## Non-Goals',
  '::: locked',
  '::: reviewed',
];
```

### Project Type Identification

```javascript
// Monorepo markers
const monorepoMarkers = [
  'packages/',
  'apps/',
  'src/modules/',
  'lerna.json',
  'pnpm-workspace.yaml',
];
```

## Options

```
/intent-init              # Interactive initialization
/intent-init --dry-run    # Show plan only, don't execute
/intent-init --minimal    # Minimal structure
/intent-init --migrate    # Attempt to migrate existing documents
```

## Integration with Other Commands

```
/intent-init              # Initialize structure
    ↓
/intent-interview         # Fill in Intent content
    ↓
/intent-review            # Approve key sections
    ↓
[Development]
    ↓
/intent-check             # Check consistency
```
