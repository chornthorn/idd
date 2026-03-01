---
name: intent-story
description: Share your IDD adoption story. Through structured interviewing, create blog posts about Intent-Driven Development experiences, lessons learned, and best practices. Supports multiple languages and formats.
---

# Intent Story

Through interview-style conversation, help users share their IDD (Intent Driven Development) adoption experiences and generate blog posts that spread the IDD methodology.

## Core Principles

- **Interview-style creation**: AI analysis + questions + user answers + AI integrates according to style
- **Real stories**: Based on user's real experiences, no fabricated cases
- **Spread IDD**: Each article naturally introduces IDD concepts and tools

## Workflow

```
/intent-story
        ↓
┌───────────────────────────────────┐
│  Phase 1: Background Analysis     │
│  - Detect input language          │
│  - Load writing style (if any)    │
│  - Understand user's IDD usage    │
│    background                     │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Phase 2: Structured Interview    │
│  - Adoption experience            │
│  - Key turning points / lessons   │
│  - Specific benefits / data       │
│  - Advice and reflections         │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Phase 3: Confirm Output Needs    │
│  - Article format / style / length│
│  - Target language versions       │
└─────────────┬─────────────────────┘
              ↓
┌───────────────────────────────────┐
│  Phase 4: Generate Article        │
│  - Full blog + social media ver.  │
│  - Multi-language versions        │
│    (on demand)                    │
│  - Naturally integrate IDD intro  │
└───────────────────────────────────┘
```

## Phase 1: Background Analysis

### Language Detection

- Determine initial writing language based on the user's primary input language
- Can ask later if other language versions are needed

### Writing Style Loading

Load by priority (if exists):
1. `~/.claude/content-profile/writing-style.md` (user-defined)
2. Use this skill's default style

### Initial Questions

Use AskUserQuestion to understand the background:

```
question: "What type of project are you using IDD for?"
header: "Project Type"
options:
  - label: "System Software / Framework"
    description: "Frameworks, libraries, infrastructure, etc."
  - label: "Web Application"
    description: "Frontend or full-stack web project"
  - label: "Mobile Application"
    description: "iOS, Android, or cross-platform"
  - label: "Other"
    description: "Other types of projects"
```

## Phase 2: Structured Interview

### Interview Dimensions

2-3 questions per round, typically 3-5 rounds to complete.

#### 1. Adoption Motivation

```
"What made you start trying IDD?"
- A) Documentation always gets outdated, looking for a better approach
- B) AI-assisted development needs better context management
- C) Team collaboration needs clearer design contracts
- D) Other reasons
```

#### 2. Turning Point

```
"During your IDD journey, was there an 'aha' moment that made it worthwhile?"
[Open-ended, collect stories]
```

#### 3. Specific Benefits

```
"What was the most noticeable change you observed?"
- A) AI code quality improved (fewer corrections needed)
- B) Architecture boundaries became clearer
- C) Documentation no longer gets outdated
- D) New members onboard faster
- E) Other
```

```
"Can you quantify the benefits? (optional)"
[Open-ended, e.g.: "The rate of AI generating usable code in one shot went from 30% to 70%"]
```

#### 4. Lessons and Challenges

```
"What challenges did you encounter or detours did you take?"
[Open-ended]
```

```
"If you could start over, what would you do differently?"
[Open-ended]
```

#### 5. Advice for Others

```
"What advice would you give to someone considering adopting IDD?"
[Open-ended]
```

### Interview Principles

- **Ask less, infer more**: Don't repeat what can be inferred from context
- **Stories first**: Encourage users to share specific experiences
- **Authenticity**: Don't fabricate cases; if the user doesn't have it, don't add it
- **Deep follow-up**: Continue digging into valuable responses

## Phase 3: Confirm Output Needs

After the interview is complete, confirm output format:

### Article Style

```
question: "What style should this article be?"
header: "Style"
options:
  - label: "Technical Sharing"
    description: "Peer-to-peer, engineer's perspective"
  - label: "Experience Summary"
    description: "Retrospective, Founder/Lead perspective"
  - label: "Getting Started Guide"
    description: "Help newcomers understand and get started"
  - label: "Case Study"
    description: "Detailed project case analysis"
```

### Article Length

```
question: "Target length?"
header: "Length"
options:
  - label: "Short (800-1200 words)"
    description: "Quick read, focused highlights"
  - label: "Medium (1500-2500 words)"
    description: "In-depth sharing"
  - label: "Long (3000+ words)"
    description: "Complete detailed case analysis"
```

### Language Versions

```
question: "Which language versions should be generated?"
header: "Language"
multiSelect: true
options:
  - label: "Chinese"
    description: "Simplified Chinese version"
  - label: "English"
    description: "English version"
  - label: "Both"
    description: "Chinese and English"
```

## Phase 4: Generate Article

### Article Structure

```markdown
# [Title: Engaging IDD Experience Title]

[Opening: Core insight/conclusion to hook the reader]

## Background / Motivation

[Why you started using IDD]

## Key Discoveries / Turning Points

[Insights and stories during usage]

## Actual Benefits

[Specific changes and quantified data]

## Lessons and Advice

[Advice for others]

## Conclusion

[Summary + IDD introduction]

---

## About IDD

[IDD overview and toolchain, see template below]
```

### IDD Introduction Template (Required)

Every article must naturally include at the end:

```markdown
---

## About IDD (Intent Driven Development)

IDD is a development methodology centered on Intent:

```
Intent → Test → Code → Sync
```

**Core Principle**: Intent is the new source code. Code review is done by AI, Intent review is done by Humans.

**Toolchain**:
- `/intent-assess` - Assess if a project is suitable for IDD
- `/intent-init` - Initialize IDD structure
- `/intent-interview` - Create Intent from ideas
- `/intent-check` - Verify code-Intent consistency

**Get Started**:
```bash
npx add-skill arcblock/idd
```

Learn more: [github.com/ArcBlock/idd](https://github.com/ArcBlock/idd)
```

### Social Media Version

Also generate a Twitter/X version:

```
[Core insight, 1-2 sentences]

[Key data/benefits]

[One-line CTA]

🔗 [Article link]

#IntentDrivenDevelopment #IDD #AIEngineering
```

### Writing Style Guide

Follow these principles (from user writing style):

| Principle | Description |
|-----------|-------------|
| Strong opinions | Don't be a fence-sitter, have a clear stance |
| Real cases | Stories come from the user, not fabricated |
| Substantial paragraphs | 4-8 sentences per paragraph, with complete arguments |
| Concise subheadings | 2-4 subheadings, rich content |
| Introspective expression | "I think..." rather than "You should..." |
| Avoid clichés | Every point needs specific insight to support it |
| No em dash | Use "-" or "," instead of "—" |

## Output Example

```markdown
# From "Documentation Always Gets Outdated" to "Intent Is Code" - My IDD Journey

Six months ago, our team faced a classic dilemma: no matter how well-written the documentation was, it was outdated in three weeks...

[Body...]

---

## About IDD

IDD (Intent Driven Development) is a development methodology centered on intent...

[Toolchain introduction...]

**Get Started**:
\`\`\`bash
git clone https://github.com/ArcBlock/idd
claude mcp add-plugin ~/path/to/idd
\`\`\`
```

## Integration with Other Commands

```
[User uses IDD for a while]
        ↓
/intent-story             # Share experience, generate blog
        ↓
Publish blog / social media
        ↓
Spread IDD methodology
```

## Notes

1. **Authenticity**: All cases and data must come from the user, nothing fabricated
2. **Natural integration**: IDD introduction should blend naturally into the ending, not be a hard advertisement
3. **User's language**: Primary language follows the user's input
4. **Style consistency**: If user writing style config exists, use it first
