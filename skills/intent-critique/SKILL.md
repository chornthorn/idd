---
name: intent-critique
description: Critical review of Intent design quality. Checks for over-engineering, YAGNI violations, premature abstraction, and simplification opportunities. Uses interactive discussion to refine design decisions.
---

# Intent Critique

Critical review of design quality. Does not check formatting, only checks "should the design be this way?"

## Core Question

Every critique answers one question: **Can this design be simpler?**

## Anti-Accretion Rule: Net Reduction

**Total line count after critique must be ≤ before.** This is a hard constraint, not a suggestion.

Critique only allows 4 operations:

| Operation | Description | Example |
|-----------|-------------|---------|
| **Delete** | Remove section or content | Remove unused plugin system design |
| **Merge** | Combine multiple sections into one | Merge Error Handling into API Constraints |
| **Split** | Move to a separate intent | §13-§18 split from ash to ash/agentic-patterns |
| **Simplify** | Express same constraints in fewer lines | Compress 5-line description to 2 lines |

**Forbidden:**
- ❌ Introduce new concepts or new sections
- ❌ Add new analysis frameworks
- ❌ Expand scope of existing sections

If something is found missing, output a gap flag, do not fill it directly:

```
Gap: Intent doesn't describe failure handling. Suggest discussing in the next interview round.
```

If new content truly needs to be added, it must:
1. First delete an equal or greater amount of old content
2. Get explicit user agreement
3. Final line count must still be ≤ initial line count

## Check Dimensions

| Dimension | Signal | Question |
|-----------|--------|----------|
| **Over-engineering** | Complex config systems, plugin architecture, multi-layer abstraction | "Is this complexity driven by current requirements?" |
| **YAGNI** | "Might need in the future", "Reserved extension points", "Support multiple..." | "If we remove this, does the MVP still work?" |
| **Premature Abstraction** | Interface with only one implementation, utility function used only once | "Do we really need this layer of abstraction now?" |
| **Boundary Issues** | Large data passing between modules, circular dependencies, unclear responsibilities | "Is this boundary natural here?" |
| **Hidden Complexity** | Descriptions that seem simple but are difficult to implement | "What is the implementation complexity behind this statement?" |

## Workflow

```
/intent-critique [path]
        ↓
┌───────────────────┐
│  Read Intent file  │
│  Understand design │
│  intent            │
└─────────┬─────────┘
          ↓
┌───────────────────┐
│  Analyze by        │
│  dimension         │
│  Identify suspect  │
│  designs           │
└─────────┬─────────┘
          ↓
┌───────────────────────────────────────┐
│  For each finding, discuss with user: │
│                                       │
│  Show issue + simplification          │
│  suggestion                           │
│  AskUserQuestion:                     │
│  - Accept simplification              │
│  - Keep original design (state reason)│
│  - Skip this item                     │
└─────────┬─────────────────────────────┘
          ↓
┌───────────────────┐
│  Summarize         │
│  critique          │
│  Optionally update │
│  Intent            │
└───────────────────┘
```

## Usage

```bash
# Review a specific Intent
/intent-critique src/core/intent/INTENT.md

# Review current directory
/intent-critique

# Analyze only, no modifications
/intent-critique --dry-run
```

## Execution Steps

### 0. Record Initial Line Count

**First step must be counting.** Before any analysis:

```bash
wc -l INTENT.md
# Record as before_lines
```

Record `before_lines` for verification of net reduction at the end of critique.

### 0.5 Convergence Check (auto-triggered when modified ≥ 3 times)

Check the number of commits for INTENT.md in git log. If ≥ 3, run convergence before the normal critique:

**4-step convergence check:**

1. **Anchor trace** — For each section ask: Can it be traced back to the anchor?
   - Cannot → Mark as deletion candidate

2. **Implementation match** — For each section ask: Is there corresponding implementation?
   - Has implementation → Keep
   - No implementation and not in plan → Mark as deletion candidate

3. **Constraint vs Analysis** — For each section ask: Is this a constraint or analysis?
   - "Three Dimensions of Chat Experience" = analysis → Simplify or move to appendix
   - "INV-1: LLM is a Pure Function" = constraint → Keep

4. **Dedup with sub-intents** — Are there sections duplicated in sub-intents?
   - Yes → Delete, leave a link

**Convergence output format:**

```markdown
## Convergence Check (triggered: 5 commits since last convergence)

### Anchor trace failures
- § Agentic Pattern Zones — cannot trace to "ASH is a shell language"
- § Echo Deprecation — migration plan, not core spec

### Unimplemented sections
- § Template Registry — no code, not in plan

### Analysis (not constraint)
- § AI Agent Compatibility Analysis — 40 lines of analysis, 0 constraints

### Duplicated in sub-intents
- § Boot Sequence — duplicates kernel/bootloader INTENT.md §2

→ Candidates for removal: 4 sections, ~280 lines
```

After convergence, proceed to normal critique flow.

### 0.7 Read Records Index

If `records/INDEX.md` exists, read the index first to understand the full decision context:

- Which interviews made which decisions
- What review feedback other AIs (ChatGPT, Gemini, etc.) have given
- Previous critique records

Read related record files in depth as needed. This solves context asymmetry — even if the original discussion happened on other AIs, there is a complete record here.

### 0.8 Devil's Advocate (Independent Assessment)

Spawn a **subagent without the current discussion context**, giving it only:
- Full text of INTENT.md
- Anchor statement

Have it independently answer 3 questions:
1. What is the biggest design risk in this intent?
2. Which sections look like analysis rather than constraints?
3. If you could only keep 3 sections, which would you keep?

Use the devil's advocate output as additional input for the critique, not adopting it directly but using it to cross-validate your own judgment.

### 1. Read and Understand Intent

Read the full Intent first to understand:
- What problem it solves
- Core design decisions
- Main components and boundaries

### 2. Scan by Dimension

#### Over-engineering Signals

```markdown
Check for:
- More than 3 configurable components
- Plugin/extension mechanisms
- More than 2 layers of abstraction
- "Support multiple X" descriptions
- Complex state machines
```

#### YAGNI Signals

```markdown
Check for:
- "Might need in the future..."
- "Reserved for..."
- "For future convenience..."
- Features only used in "advanced scenarios"
- Abstractions without concrete use cases
```

#### Premature Abstraction Signals

```markdown
Check for:
- Interface/protocol with only one implementation
- Public function used only once
- Base class with only one subclass
- Unnecessary factory/strategy patterns
```

#### Boundary Issue Signals

```markdown
Check for:
- More than 5 parameters passed between modules
- A depends on B, B also depends on A
- A single change requires modifying multiple modules
- Vague or overlapping responsibility descriptions
```

### 3. Generate Findings List

For each finding, prepare:
- **Location**: Specific section in Intent
- **Issue**: Brief description of the problem
- **Simplification Suggestion**: Specific alternative approach
- **Risk**: Potential cost of simplification

### 4. Interactive Discussion

Use AskUserQuestion for each finding:

```
Finding 1/N: Over-engineering

Section: ## Plugin System
Issue: Defines complete plugin loading, lifecycle, and sandbox mechanisms, but currently only has 2 built-in features.

Simplification Suggestion: Remove plugin system, hardcode these 2 features directly.
Add back when needed in the future.

Risk: If third-party plugins are needed soon, redesign required.

---
AskUserQuestion:
- question: "Accept this simplification?"
- options:
  - "Accept simplification" - Remove plugin system design
  - "Keep original design" - Please state the specific scenario
  - "Skip" - Decide later
```

### 5. Summary Report

```markdown
# Intent Critique Report

## File: src/core/intent/INTENT.md

## Statistics
- Issues found: 5
- Simplifications accepted: 3
- Original designs kept: 1
- Skipped: 1

## Accepted Simplifications

### 1. Remove Plugin System
- Original design: Full plugin architecture
- Simplified to: Hardcoded 2 features
- Affected sections: ## Plugin System, ## Architecture

### 2. Merge Config Layers
...

## Kept Designs

### 1. Multi-data-source Support
- Reason: User explicitly stated needed for v1.1
- Suggestion: Note the specific timeline in Intent

## Skipped Items
...

## Next Steps
- [ ] Update Intent file (if auto-update chosen)
- [ ] Run /intent-validate to check format
- [ ] Run /intent-review to re-approve affected sections
```

### 6. Optional: Update Intent

If user agrees, automatically update Intent:
- Delete simplified sections
- Add simplification notes to change log
- Mark affected sections back to `draft`

### 7. Verify Net Reduction (Required)

After updating Intent, immediately verify:

```bash
wc -l INTENT.md
# Record as after_lines
```

**Decision rules:**

```
if after_lines > before_lines:
    ❌ Refuse to save. Rollback changes.
    Report:
      before: {before_lines} lines
      after:  {after_lines} lines
      delta:  +{after_lines - before_lines} lines
      "Critique must net-reduce. Review changes and remove additions."

if after_lines <= before_lines:
    ✅ Save successful.
    Report:
      before: {before_lines} lines
      after:  {after_lines} lines
      reduced: {before_lines - after_lines} lines ({percentage}%)
```

### 8. Save Critique Record

Save the complete critique report to `records/critique-{date}.md`, update `records/INDEX.md`:

```markdown
| 2026-02-06 | critique | — | Convergence + critique, removed §13-§15 (-180 lines), kept §2 |
```

The record includes: convergence results, devil's advocate output, decisions for each finding, net line count change.

## Discussion Techniques

### Questioning Approach

**Good questions**:
- "If we remove X, does the system still work?"
- "Is there a specific scenario that requires X?"
- "Is the complexity of X worth it?"

**Avoid**:
- Presupposing answers
- Overly theoretical discussions
- Asking multiple unrelated questions at once

### When to Accept "Keep"

When the user provides:
- Specific usage scenarios
- Clear timeline ("needed next month")
- External constraints ("required by partner")

Record the rationale, and suggest noting it in the Intent as well.

## Integration with Other Tools

```
intent-interview → intent-validate → intent-critique → intent-review
                                          ↑
                                     Can be invoked independently
                                     Whenever you want to reflect on design
```

## What It Does NOT Do

- ❌ Does not check formatting (intent-validate's responsibility)
- ❌ Does not check consistency (intent-sync's responsibility)
- ❌ Does not mark approval status (intent-review's responsibility)
- ❌ Does not make decisions automatically (must discuss with user)
