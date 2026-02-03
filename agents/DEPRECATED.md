# Deprecated Agents

These agents are **deprecated** as of 2026-01. Their capabilities have been absorbed into the plan.md format.

## Why Deprecated

The IDD workflow has evolved:

**Before**: Runtime agents generated tests and orchestrated execution
```
/intent-build-now
    → idd-task-execution-master (分解 phases)
    → idd-test-master (生成测试)
    → idd-code-guru (实现代码)
    → idd-e2e-test-queen (E2E 测试)
```

**After**: plan.md contains complete test matrix, TaskSwarm executes
```
/intent-plan
    → Generates plan.md (complete test matrix)
    → Generates TASK.yaml (status tracking)

/swarm run
    → Executes TDD phases directly from plan.md
    → No intermediate agents needed
```

## Migration

| Old Agent | Replacement |
|-----------|-------------|
| `idd-task-execution-master` | plan.md Phase structure |
| `idd-test-master` | plan.md 6-category test matrix |
| `idd-code-guru` | Claude direct execution |
| `idd-e2e-test-queen` | plan.md E2E Gate sections |

## Safe to Delete

These files can be safely removed:
- `idd-task-execution-master.md`
- `idd-test-master.md`
- `idd-code-guru.md`
- `idd-e2e-test-queen.md`

The validation/sync agents remain useful:
- `intent-validate.md` - Keep (validates Intent structure)
- `intent-sync.md` - Keep (syncs implementation back to Intent)
- `intent-audit.md` - Keep (project-level health check)
