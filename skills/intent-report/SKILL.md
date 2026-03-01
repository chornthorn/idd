---
name: intent-report
description: Generate human-readable report from Intent files. Converts technical Intent specs into readable documents for stakeholders, team members, or documentation. Supports multiple output formats.
---

# Intent Report

Convert technical Intent files into human-readable report documents.

## Purpose

- **For Stakeholders**: Project overview, progress, key decisions
- **For New Members**: Quickly understand project architecture and design rationale
- **For Documentation**: Generate READMEs, design docs, architecture descriptions
- **For Meetings**: Project presentations, technical reviews

## Workflow

```
/intent-report [options]
        ↓
┌───────────────────────────────────┐
│  Read Intent files                │
│  - Project-level + module-level   │
│  - Parse structure and metadata   │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Determine report type            │
│  - overview / architecture /      │
│    progress / full                │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Generate report                  │
│  - Restructure content            │
│  - Convert technical language     │
│  - Add visualizations             │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Output                           │
│  - Markdown / HTML / PDF          │
│  - Write to file or display       │
└───────────────────────────────────┘
```

## Report Types

### 1. Overview

```
/intent-report --type overview
```

One-page project overview, suitable for quick understanding:

```markdown
# [Project] Overview

## What is this?
[One-line description]

## Problem
[Problem to solve]

## Solution
[Solution overview]

## Architecture
[Simplified architecture diagram]

## Key Modules
| Module | Purpose |
|--------|---------|

## Status
[Current status and next steps]
```

### 2. Architecture

```
/intent-report --type architecture
```

Detailed architecture document, suitable for technical review:

```markdown
# [Project] Architecture

## System Overview
[Architecture diagram]

## Module Dependencies
[Dependency graph]

## Data Flow
[Data flow diagram]

## Key Design Decisions
| Decision | Choice | Rationale |

## Boundary Rules
[Module boundary rules]

## API Reference
[Core API list]
```

### 3. Progress

```
/intent-report --type progress
```

Project progress report, suitable for presentations:

```markdown
# [Project] Progress Report

> Generated: YYYY-MM-DD

## Intent Coverage
[Coverage chart]

## Module Status
| Module | Intent | Impl | Status |
|--------|--------|------|--------|
| core   | ✓      | 80%  | 🟡     |

## Recent Updates
[Recent Intent changes]

## Approval Status
- Locked: N sections
- Reviewed: M sections
- Draft: K sections

## Blockers & Risks
[Risks and blockers]

## Next Steps
[Next steps plan]
```

### 4. Full

```
/intent-report --type full
```

Complete technical document, including all content.

## Output Formats

### Markdown (Default)

```
/intent-report -o report.md
```

### HTML

```
/intent-report --format html -o report.html
```

Styled HTML document, viewable directly in a browser.

### Console (Direct Display)

```
/intent-report
```

When no output file is specified, display directly in the terminal.

## Usage Examples

### Generate Project Overview

```
/intent-report --type overview
```

### Generate Architecture Doc for New Members

```
/intent-report --type architecture -o docs/ARCHITECTURE.md
```

### Generate Progress Report for Stakeholders

```
/intent-report --type progress -o reports/progress-2026-01.md
```

### Generate Single Module Report

```
/intent-report src/core/ --type full
```

## Content Conversion Rules

### Technical Language → Human Language

| In Intent | In Report |
|-----------|-----------|
| `## Responsibilities` | "What this module does" |
| `## Non-Goals` | "Out of scope" |
| `## Constraints` | "Constraints & Rules" |
| `::: locked` | "Core Architecture (frozen)" |
| `::: reviewed` | "Approved Design" |
| `::: draft` | "Work in Progress" |

### ASCII Diagrams → Visualizations

- Keep ASCII diagrams (good compatibility)
- Optional: Convert to Mermaid diagrams (`--mermaid` option)

## Integration with Other Commands

```
/intent-init              # Initialize
    ↓
/intent-interview         # Create Intent
    ↓
/intent-review            # Approve
    ↓
/intent-report            # ← Generate report (this command)
    ↓
Share with stakeholders / team
```

## Advanced Options

```
/intent-report
  --type <type>           # overview | architecture | progress | full
  --format <format>       # markdown | html
  --output <path>         # Output file path
  --module <path>         # Specify module
  --include-draft         # Include draft sections
  --mermaid               # Convert to Mermaid diagrams
  --lang <lang>           # Output language (en | zh)
```
