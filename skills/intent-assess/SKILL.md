---
name: intent-assess
description: Assess if IDD fits your project and learn about Intent-Driven Development. Use /intent-assess to evaluate project suitability or /intent-assess --learn for IDD education.
---

# Intent Assess

Assess whether a project is suitable for IDD and educate on the IDD methodology.

## Two Modes

### 1. Assessment Mode

```
/intent-assess
```

Analyze the current project and assess whether IDD adoption is appropriate.

### 2. Learning Mode

```
/intent-assess --learn
```

Educate on the IDD methodology, explain core concepts.

---

## Assessment Mode

### Workflow

```
/intent-assess
        ↓
┌───────────────────────────────────┐
│  Phase 1: Project Analysis        │
│  - Project type identification    │
│  - Codebase size                  │
│  - Existing documentation status  │
│  - Team collaboration model       │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Phase 2: Fit Assessment          │
│  - Calculate match score          │
│  - Identify strengths & challenges│
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Phase 3: Recommendations         │
│  - Whether to recommend IDD       │
│  - How to get started             │
│  - Alternatives (if not suitable) │
└───────────────────────────────────┘
```

### Assessment Dimensions

| Dimension | Favorable for IDD | Unfavorable for IDD |
|-----------|-------------------|---------------------|
| **Project Type** | System software, frameworks, libraries | Simple scripts, one-off projects |
| **Code Size** | Medium-large (>5k LOC) | Small (<1k LOC) |
| **Team Collaboration** | Multi-person, AI-assisted | Solo development |
| **Architecture Complexity** | Multi-module, needs boundaries | Single module, simple structure |
| **Iteration Style** | Continuous iteration, long-term maintenance | One-time delivery |
| **AI Tool Usage** | Using Claude/Copilot | Purely manual development |

### Assessment Report Example

```markdown
# IDD Assessment Report

> Project: ainecore
> Date: 2026-01-19

## Fit Score: 85/100 ⭐⭐⭐⭐

## Project Characteristics

| Characteristic | Current Status | IDD Fit |
|----------------|---------------|---------|
| Project Type | Framework/Platform | ✅ High |
| Code Size | ~15k LOC | ✅ High |
| Module Count | 12 modules | ✅ High |
| Team Size | 3 people + AI | ✅ High |
| Existing Docs | Partial | 🟡 Medium |

## Strengths

- ✅ Multi-module architecture needs clear boundary definitions
- ✅ AI-assisted development, Intent can guide AI
- ✅ Long-term maintenance project, high documentation value
- ✅ Has some existing design docs, can be migrated

## Challenges

- ⚠️ Need to establish Intent writing habits
- ⚠️ Existing code needs Intent supplements
- ⚠️ Team needs to learn IDD methodology

## Recommendations

### Recommended: Adopt IDD ✅

This project is well-suited for IDD:
1. Multi-module architecture needs clear boundaries and contracts
2. AI-assisted development can directly use Intent as context
3. High long-term maintenance value

### Getting Started Suggestions

1. **Start with core modules**
   - Write Intent for `src/core/` first
   - Establish Intent templates and standards

2. **Gradual rollout**
   - New features must have Intent first
   - Existing code gets Intent gradually

3. **Tool integration**
   - Install IDD plugin
   - Configure CI/CD integration

### Expected Benefits

- 🎯 AI coding efficiency improvement ~30%
- 🎯 Clearer architecture boundaries
- 🎯 Faster new member onboarding
- 🎯 Reduced "outdated documentation" issues
```

---

## Learning Mode

### Interactive Teaching

```
/intent-assess --learn
```

Teach IDD through Q&A:

```
┌───────────────────────────────────┐
│  Welcome to IDD!                  │
│                                   │
│  I will introduce:                │
│  1. What is IDD                   │
│  2. IDD vs TDD vs SDD            │
│  3. Intent file structure         │
│  4. Real examples                 │
│                                   │
│  Which topic would you like to    │
│  start with?                      │
└───────────────────────────────────┘
```

### Core Concept Explanations

#### 1. What is IDD

```
Development methodology evolution:

Traditional:  Code → Test → Docs
              (Documentation often gets outdated)

SDD:          Spec → Code → Test
              (Specs scattered, hard to maintain)

TDD:          Test → Code → Docs
              (Tests can't capture design rationale)

IDD:          Intent → Test → Code → Sync
              (Intent as the single source of truth)
```

#### 2. Intent Three-Layer Structure

```
┌─────────────────────────────────────┐
│  Layer 1: Structure                 │
│  - Directory structure, data        │
│    structures, module relationships │
│  - ASCII diagrams preferred         │
├─────────────────────────────────────┤
│  Layer 2: Constraints               │
│  - Dependency directions, boundary  │
│    rules, invariants                │
│  - Can be converted to test         │
│    assertions                       │
├─────────────────────────────────────┤
│  Layer 3: Examples                  │
│  - Input → Output examples         │
│  - Edge cases                       │
└─────────────────────────────────────┘
```

#### 3. IDD vs SDD Comparison

```markdown
| Dimension | SDD | IDD |
|-----------|-----|-----|
| Organization | By type (feature/UX/tech) | By module |
| Core Medium | Text descriptions | Structure diagrams |
| Granularity | Granular User Stories | Complete Patterns |
| Task Management | Separate Task files | AI autonomously decomposes |
| LLM Friendliness | Requires context assembly | Understands completely at once |
```

#### 4. Real Examples

Show a real Intent file and explain each part's role.

### Quick Reference

```
/intent-assess --learn --topic <topic>
```

Available topics:
- `what` - What is IDD
- `vs-sdd` - IDD vs SDD comparison
- `vs-tdd` - IDD vs TDD comparison
- `structure` - Intent file structure
- `workflow` - IDD workflow
- `approval` - Section approval mechanism
- `best-practices` - Best practices

---

## Integration with Other Commands

```
/intent-assess                # Assess project
    ↓ (if suitable)
/intent-assess --learn        # Learn IDD
    ↓
/intent-init                  # Initialize IDD
    ↓
/intent-interview             # Create Intent
```
