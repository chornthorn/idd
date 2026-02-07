# Intent Data Format Standard

> Machine-readable computed views of Intent health, dependencies, and code mappings.
> Version: 1.0

## Overview

IDD generates structured data files that serve as the interface contract between skill producers (IDD) and consumers (TeamSwarm, dashboards, CI). These files are **computed views** — never hand-edited, always regenerated from source.

**Analogy:** Like `package-lock.json` — derived from source, deterministic output, safe to regenerate.

---

## File Location

All data files live in `intent/_data/`:

```
intent/
├── _data/                    # Computed data (generated, never hand-edited)
│   ├── intent-health.json    # Per-intent health metrics
│   ├── intent-graph.json     # Dependency graph (nodes + edges)
│   └── intent-map.json       # Intent → code mappings
├── INTENT.md
└── ...
```

**Gitignore:** Optional. Files can be regenerated from source at any time. Projects may choose to commit them for CI caching or dashboard access.

**Generation:** Run `intent-audit` agent with `outputFormat: "json"` to generate `intent-health.json` and `intent-graph.json`. Run `intent-sync` agent to generate `intent-map.json`.

---

## Schemas

### `intent-health.json` — Per-Intent Health Metrics

Reports the health status of every intent in the project, including anti-accretion checks.

```json
{
  "generated": "2026-02-06T10:00:00Z",
  "version": "1.0",
  "intents": {
    "<intent-id>": {
      "path": "intent/kernel/proc/INTENT.md",
      "lines": 413,
      "budget_status": "healthy | warning | blocked",
      "anchor": "Process lifecycle management for AOS kernel",
      "assumes": ["kernel/afs-scoping"],
      "commits_since_convergence": 5,
      "last_modified": "2026-02-05",
      "last_convergence": "2026-02-03",
      "convergence_due": false,
      "sections": 12,
      "analysis_sections": 2,
      "approval": {
        "locked": 3,
        "reviewed": 5,
        "draft": 4
      }
    }
  },
  "summary": {
    "total": 42,
    "healthy": 30,
    "warning": 8,
    "blocked": 4,
    "orphans": 15,
    "stale_refs": 4,
    "convergence_overdue": 6
  }
}
```

**Field definitions:**

| Field | Type | Description |
|-------|------|-------------|
| `generated` | ISO 8601 | Generation timestamp |
| `version` | string | Schema version |
| `intents` | object | Map of intent ID → metrics |
| `intents.<id>.path` | string | Relative path to INTENT.md |
| `intents.<id>.lines` | number | Total line count (per Size Budget rules) |
| `intents.<id>.budget_status` | enum | `healthy` (≤300), `warning` (300–500), `blocked` (>500) |
| `intents.<id>.anchor` | string | Anchor statement text |
| `intents.<id>.assumes` | string[] | Intent IDs from `Assumes:` tag |
| `intents.<id>.commits_since_convergence` | number | Commits touching this intent since last convergence |
| `intents.<id>.last_modified` | date | Last modification date |
| `intents.<id>.last_convergence` | date \| null | Last convergence check date |
| `intents.<id>.convergence_due` | boolean | True if commits ≥ 3 since last convergence |
| `intents.<id>.sections` | number | Total section count |
| `intents.<id>.analysis_sections` | number | Sections classified as analysis (not constraint) |
| `intents.<id>.approval` | object | Approval status counts |
| `summary.total` | number | Total intent count |
| `summary.healthy` | number | Intents with budget_status = healthy |
| `summary.warning` | number | Intents with budget_status = warning |
| `summary.blocked` | number | Intents with budget_status = blocked |
| `summary.orphans` | number | Intents with no Assumes edges (in or out) |
| `summary.stale_refs` | number | Assumes references pointing to deleted/moved intents |
| `summary.convergence_overdue` | number | Intents with convergence_due = true |

**Intent ID:** Relative path from `intent/` to the directory containing `INTENT.md`, using `/` as separator. Example: `kernel/proc` for `intent/kernel/proc/INTENT.md`.

---

### `intent-graph.json` — Dependency Graph

Represents the full dependency graph as nodes and edges, suitable for visualization and cycle detection.

```json
{
  "generated": "2026-02-06T10:00:00Z",
  "version": "1.0",
  "nodes": [
    {
      "id": "kernel/proc",
      "anchor": "Process lifecycle management for AOS kernel",
      "lines": 413,
      "status": "warning"
    }
  ],
  "edges": [
    {
      "from": "kernel/proc",
      "to": "kernel/afs-scoping",
      "type": "assumes"
    },
    {
      "from": "ash",
      "to": "kernel/proc",
      "type": "hard_ref"
    }
  ],
  "orphans": ["surfaces"],
  "stale_refs": [
    {
      "from": "ash",
      "to": "kernel/echo",
      "reason": "target deleted"
    }
  ]
}
```

**Field definitions:**

| Field | Type | Description |
|-------|------|-------------|
| `nodes` | array | All intents as graph nodes |
| `nodes[].id` | string | Intent ID |
| `nodes[].anchor` | string | Anchor statement |
| `nodes[].lines` | number | Line count |
| `nodes[].status` | enum | Budget status: `healthy`, `warning`, `blocked` |
| `edges` | array | All dependency edges |
| `edges[].from` | string | Source intent ID |
| `edges[].to` | string | Target intent ID |
| `edges[].type` | enum | `assumes` (from Assumes: tag) or `hard_ref` (from inline references) |
| `orphans` | string[] | Intent IDs with no edges in or out |
| `stale_refs` | array | Broken references |
| `stale_refs[].from` | string | Source intent ID |
| `stale_refs[].to` | string | Referenced intent ID (missing) |
| `stale_refs[].reason` | string | Why the reference is stale |

**Edge extraction:**
- `assumes` edges: parsed from `> Assumes:` tag in INTENT.md anchor block
- `hard_ref` edges: detected from inline references to other intent IDs in the document body

---

### `intent-map.json` — Intent → Code Mappings

Maps intent sections to source files and test files. Used by `intent-sync` to detect drift between intent and implementation.

```json
{
  "generated": "2026-02-06T10:00:00Z",
  "version": "1.0",
  "mappings": {
    "<intent-id>": {
      "sections": {
        "§3.1": {
          "sources": [
            {
              "path": "chat-history-backend.ts",
              "lines": [1, 32],
              "confidence": "annotated | heuristic"
            }
          ],
          "tests": [
            {
              "path": "chat-history.test.ts",
              "invariants": ["INV-CH-4"]
            }
          ]
        }
      },
      "unmapped_sections": ["§5"],
      "unmapped_sources": []
    }
  }
}
```

**Field definitions:**

| Field | Type | Description |
|-------|------|-------------|
| `mappings` | object | Map of intent ID → mapping data |
| `sections` | object | Map of section ref → source/test links |
| `sections.<ref>.sources` | array | Source files implementing this section |
| `sources[].path` | string | Relative file path |
| `sources[].lines` | [number, number] | Line range [start, end] |
| `sources[].confidence` | enum | `annotated` (explicit marker) or `heuristic` (inferred) |
| `sections.<ref>.tests` | array | Test files covering this section |
| `tests[].path` | string | Relative test file path |
| `tests[].invariants` | string[] | Invariant IDs tested |
| `unmapped_sections` | string[] | Sections with no source mapping |
| `unmapped_sources` | string[] | Source files not traced to any section |

**Confidence levels:**
- `annotated`: Source file contains explicit intent reference (e.g., `// Intent: kernel/proc §3.1`)
- `heuristic`: Mapping inferred from naming conventions, proximity, or content analysis

---

## Design Principles

1. **Computed, not authored** — All data files are generated from source. No manual editing.
2. **Deterministic** — Same input always produces same output (except `generated` timestamp).
3. **Schema versioned** — `version` field enables consumers to handle format changes.
4. **Flat intent IDs** — Use `/`-separated relative paths as canonical identifiers.
5. **Consumer-agnostic** — IDD defines the format; TeamSwarm, dashboards, and CI interpret it.

---

## Related Documents

- [intent-standard.md](intent-standard.md) — `Assumes:` tag format, Size Budget rules
- [intent-approval.md](intent-approval.md) — Section approval mechanism (locked/reviewed/draft)

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-02-06 | Initial version — health, graph, map schemas |
