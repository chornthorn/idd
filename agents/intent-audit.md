---
name: intent-audit
description: Project-level Intent health check. Use for periodic audits, onboarding, or before major releases. Reports coverage, freshness, approval status, dependency consistency, and boundary violations.
tools: Read, Grep, Glob
model: inherit
---

# Intent Audit Agent

> 项目级 Intent 健康检查

## 触发场景

- 定期检查（每周/每月）
- 新成员 onboard 时了解项目状态
- 大版本发布前

## 输入

```
{
  "projectRoot": "/path/to/project",
  "outputFormat": "markdown" | "json"
}
```

## 范围

- Scan all `INTENT.md` files under `intent/` and module-level `intent/` directories
- **Skip `intent/_archive/`** — archived intents are excluded from all checks
- **Skip `intent/_data/`** — generated data files, not intents
- Read frontmatter `status` field when present; default to `active` if omitted

## 检查维度

### 1. 覆盖率

```
模块 Intent 覆盖率：

src/
├── core/
│   └── intent/INTENT.md        ✅
├── platforms/
│   ├── web/
│   │   └── intent/             ❌ 缺失
│   ├── cli/
│   │   └── intent/INTENT.md    ✅
│   └── desktop/
│       └── intent/             ❌ 缺失
└── chambers/
    ├── terminal/
    │   └── intent/INTENT.md    ✅
    ├── portal/
    │   └── intent/INTENT.md    ✅
    └── taskboard/
        └── intent/             ❌ 缺失

覆盖率: 4/7 (57%)
```

### 2. 新鲜度

```
Intent 更新状态：

| 模块 | 最后更新 | 代码最后修改 | 状态 |
|------|---------|-------------|------|
| core | 2026-01-19 | 2026-01-19 | ✅ 同步 |
| terminal | 2026-01-15 | 2026-01-18 | ⚠️ 可能过时 |
| cli | 2026-01-10 | 2026-01-19 | ❌ 严重过时 |
```

### 3. 审批状态

```
Section 审批统计：

项目总计：
├── 🔒 LOCKED: 12 sections
├── ✓ REVIEWED: 28 sections
└── 📝 DRAFT: 45 sections

审批率: 47% (40/85)

需要关注：
- core/intent/INTENT.md 有 8 个 draft sections
- cli/intent/INTENT.md 全部未审批
```

### 4. 依赖一致性

检查 `intent/architecture/DEPENDENCIES.md` 与实际 import 的一致性：

```
依赖规则检查：

声明的依赖：
  API → Chamber ✓
  API → Deploy ✓
  Deploy → Chamber ✓

实际 import：
  API → Chamber ✅ 一致
  API → Deploy ✅ 一致
  API → Router ❌ 未声明！
  Deploy → Chamber ✅ 一致
```

### 5. 边界违规

扫描代码，检查 `intent/architecture/BOUNDARIES.md` 中的规则：

```
边界规则检查：

规则 1: "上层不直接拼接 appsDir 路径"
├── routes/apps.js:45      ❌ join(appsDir, name)
├── routes/apps.js:78      ❌ join(appsDir, name, 'branches')
└── routes/deploy.js:23    ✅ 使用 chamber.getPath()

规则 2: "deploy 不直接 import router"
└── 全部通过 ✅

违规总计: 2 处
```

### 6. Size Budget

Per-intent line count vs tiered thresholds (from `intent-standard.md` Anti-Accretion Rules):

```
Size Budget 检查：

| Intent | Lines | Status |
|--------|-------|--------|
| kernel/proc | 413 | ⚠️ Warning (300–500) |
| kernel/afs-scoping | 187 | ✅ Healthy (≤300) |
| ash | 612 | ❌ Blocked (>500) — must run /intent-critique |

Budget Summary: 30 healthy, 8 warning, 4 blocked
```

**Thresholds:** ≤ 300 healthy, 300–500 warning, > 500 blocked.

### 7. Convergence Overdue

Detects intents with ≥ 3 commits since last convergence check (critique/sync):

```
Convergence 检查：

| Intent | Commits Since | Last Convergence | Status |
|--------|---------------|------------------|--------|
| kernel/proc | 5 | 2026-02-03 | ❌ Overdue (≥3) |
| ash | 1 | 2026-02-05 | ✅ OK |
| surfaces | 4 | null | ❌ Never converged |

Convergence Overdue: 6 intents
```

**Detection method:** Count git commits touching `INTENT.md` since the most recent `records/critique-*.md` or `records/sync-*.md` entry in `records/INDEX.md`.

### 8. Analysis Ratio

Classifies sections as constraint (testable rule) vs analysis (background/exploration). High analysis ratio suggests the intent needs convergence:

```
Analysis Ratio 检查：

| Intent | Total Sections | Analysis | Ratio | Status |
|--------|---------------|----------|-------|--------|
| kernel/proc | 12 | 2 | 17% | ✅ OK |
| ash | 15 | 8 | 53% | ⚠️ High analysis content |
```

**Heuristic:** Sections with words like "Background", "Context", "Exploration", "Notes", "Discussion", "Options", "Alternatives" are classified as analysis. Sections with "API", "Constraints", "Rules", "Structure", "Invariants", "Parameters" are classified as constraint.

### 9. Orphan Detection

Intents with no `Assumes:` edges in or out — not depended on and not depending on anything:

```
Orphan 检查：

Orphan intents (no dependency edges):
- surfaces
- legacy/migration
- experimental/playground

Orphans: 3 intents (of 42 total)
```

**Note:** Root-level project intent and leaf intents may legitimately have no edges. Flag for review, not necessarily an error.

### 10. Stale References

`Assumes:` tags pointing to deleted or moved intents:

```
Stale Reference 检查：

| From | References | Status |
|------|-----------|--------|
| ash | kernel/echo | ❌ Target deleted |
| api/gateway | auth/legacy | ❌ Target moved to auth/v2 |

Stale References: 2
```

**Detection:** For each `Assumes:` target, verify the corresponding `intent/<id>/INTENT.md` exists.

## 输出

### Markdown 报告

```markdown
# Intent Audit Report

> 项目: ainecore
> 日期: 2026-01-19
> 执行者: intent-audit agent

## 总览

| 指标 | 值 | 状态 |
|------|-----|------|
| 模块覆盖率 | 57% | ⚠️ |
| 审批率 | 47% | ⚠️ |
| 依赖一致性 | 92% | ✅ |
| 边界合规 | 85% | ⚠️ |
| Size Budget | 30/42 healthy | ⚠️ |
| Convergence | 6 overdue | ❌ |
| Orphans | 15 intents | ⚠️ |
| Stale Refs | 4 broken | ❌ |

## 健康分数: 72/100

## 优先行动

### P0 - 立即处理
1. 修复 2 处边界违规 (routes/apps.js)
2. 补充 cli 模块的 Intent 审批

### P1 - 本周处理
1. 为 web, desktop, taskboard 添加 Intent
2. 更新 cli Intent（严重过时）

### P2 - 计划中
1. 提高审批率到 70%
2. 添加 DEPENDENCIES.md

## 详细报告

[展开各维度详情...]
```

### JSON 报告 (Structured Output)

When `outputFormat: "json"`, generate `intent/_data/intent-health.json` and `intent/_data/intent-graph.json` following the schemas defined in [intent-data-format.md](../docs/intent-data-format.md).

**`intent-health.json`** — per-intent metrics including anti-accretion checks:

```json
{
  "generated": "2026-02-06T10:00:00Z",
  "version": "1.0",
  "intents": {
    "kernel/proc": {
      "path": "intent/kernel/proc/INTENT.md",
      "lines": 413,
      "budget_status": "warning",
      "anchor": "Process lifecycle management for AOS kernel",
      "assumes": ["kernel/afs-scoping"],
      "commits_since_convergence": 5,
      "last_modified": "2026-02-05",
      "last_convergence": "2026-02-03",
      "convergence_due": true,
      "sections": 12,
      "analysis_sections": 2,
      "approval": { "locked": 3, "reviewed": 5, "draft": 4 }
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

**`intent-graph.json`** — dependency graph:

```json
{
  "generated": "2026-02-06T10:00:00Z",
  "version": "1.0",
  "nodes": [
    { "id": "kernel/proc", "anchor": "...", "lines": 413, "status": "warning" }
  ],
  "edges": [
    { "from": "kernel/proc", "to": "kernel/afs-scoping", "type": "assumes" }
  ],
  "orphans": ["surfaces"],
  "stale_refs": [
    { "from": "ash", "to": "kernel/echo", "reason": "target deleted" }
  ]
}
```

**Output location:** `intent/_data/` directory (created if not exists).

### Legacy JSON 报告

For backward compatibility, the inline JSON report is still supported:

```json
{
  "project": "ainecore",
  "date": "2026-01-19",
  "score": 72,
  "coverage": {
    "total": 7,
    "covered": 4,
    "percentage": 57
  },
  "approval": {
    "locked": 12,
    "reviewed": 28,
    "draft": 45,
    "percentage": 47
  },
  "violations": [
    {
      "file": "routes/apps.js",
      "line": 45,
      "rule": "no-direct-path",
      "severity": "error"
    }
  ],
  "recommendations": [...]
}
```

## 集成

### CI/CD 集成

```yaml
# .github/workflows/intent-audit.yml
name: Intent Audit
on:
  schedule:
    - cron: '0 9 * * 1'  # 每周一早上
jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Intent Audit
        run: claude --agent intent-audit --output audit-report.md
      - name: Upload Report
        uses: actions/upload-artifact@v4
        with:
          name: intent-audit-report
          path: audit-report.md
```

### 阈值告警

```json
{
  "thresholds": {
    "coverage": 70,
    "approval": 60,
    "violations": 0
  },
  "failOnThreshold": true
}
```
