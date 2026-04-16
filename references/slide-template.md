# Slide Structure Templates

## Tech Mode (6-part, 3-4 slides per project)

```markdown
---
# Slide: {project} — 背景 & 问题
## 背景
{narrative.goal}
## 本周核心问题
{narrative.problems}

---
# Slide: {project} — 关键改动 & 技术方案
## 关键改动
{narrative.key_changes}
## 技术方案
{narrative.technical_approach}

---
# Slide: {project} — 成果 & 下一步
## 当前成果
{narrative.result}
## 风险 & 下一步
{narrative.risk_and_next}
```

## Report Mode (4-part, 2-3 slides per project)

```markdown
---
# Slide: {project} — 目标 & 关键改动
## 本周目标
{narrative.goal}
## 关键改动
{narrative.key_changes}

---
# Slide: {project} — 成果 & 下一步
## 当前成果
{narrative.result}
## 下一步
{narrative.risk_and_next}
```

## Common Slides

### Title (Slide 1)

```markdown
---
# 标题页
## 本周工作汇报
**日期：** YYYY-MM-DD | **范围：** project-a / project-b / project-c
```

### Overview (Slide 2)

```markdown
---
# 本周总览
- **project-a：** {narrative.goal} (one sentence)
- **project-b：** {narrative.goal} (one sentence)
- **project-c：** {narrative.goal} (one sentence)
```

Natural cross-project themes are fine here when they genuinely emerge, but don't force connections that don't exist.

### Summary (Final Slide)

```markdown
---
# 总结 & 下一步
## 全局总结
| 项目 | 状态 | 关键进展 |
|------|------|----------|
| project-a | {current_status} | {key_changes summary} |
| project-b | {current_status} | {key_changes summary} |

## 各项目下一步
- **project-a：** {narrative.risk_and_next}
- **project-b：** {narrative.risk_and_next}
```

## Slide Budget by Priority

| Priority | Slides | Depth |
|----------|--------|-------|
| Core (1st) | 3-4 | Full narrative |
| Supporting (2nd) | 2-3 | Focused on key changes |
| Exploratory (3rd+) | 1-2 | Status update only |
