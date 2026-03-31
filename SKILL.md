---
name: weekly-multi-project-ppt-lite
description: Use when generating weekly PPT outlines from multiple projects' git commit history, when user says "生成周报PPT", "多项目周报", "weekly report", "周报", or when asked to summarize git history across projects into presentation format
---

# Weekly Multi-Project PPT Lite

## Overview

Convert **multiple projects' weekly git commits** into a structured Markdown PPT outline. Each project gets an independent narrative (Problem → Solution → Result) using a unified template. The overview slide can note natural cross-project themes, but each project's slides stay self-contained.

## When to Use

- Multiple projects running in parallel
- User provides git commit history and wants a presentation-style report
- Projects have independent goals

**Not for:** Single-project reports, actual .pptx generation (use weekly-report-ppt), non-git-based reporting.

## Inputs

| Input | Required | How to get |
|-------|----------|-----------|
| Git logs per project | Yes | User provides, or run `git log --oneline --since="last week"` per repo |
| Project priority order | Recommended | Infer from commit volume if not specified |
| Mode: `tech` / `report` | Optional (default: `tech`) | Ask user or infer from context |

**Mode controls output depth:**
- `tech` — Full 6-part narrative per project, includes architecture decisions and design choices
- `report` — Condensed 4-part (Goal → Changes → Impact → Next), focuses on outcomes over implementation

## Pipeline

```
git analyze → abstract → storyline → stitch
```

Complete each step before moving to the next. The abstraction step (Step 2) is where raw commits become engineering insight — skipping it produces the "commit流水账" (commit dump) that no one wants to read in a presentation.

### Step 1: Git Analysis

**Keep:** `feat/feature`, `fix`, `refactor`, `perf`. **Drop:** `chore`, `docs`, `style`.

Output: `change_blocks` per project with `{ type, module, summary, files }`.

If a project has **0 change_blocks** after filtering (all chores/docs), it's a maintenance week — note it on the overview slide with a brief status, don't create dedicated project slides for empty content.

### Step 2: Change Abstraction

Group related commits into abstracted engineering changes. This is the step that separates a useful report from a commit dump.

**Example:** 3 commits — "add auto-layout" + "fix overlap" + "add positioning" → ONE key_change: "Built scene composition system with auto-layout and dense-panel overlap resolution"

**Per-project output:**

| Field | Limit | Captures |
|-------|-------|----------|
| `weekly_goal` | 1 sentence | What the project aimed to achieve this week |
| `core_problems` | ≤ 2 | Pain points or blockers addressed |
| `key_changes` | ≤ 4 | Abstracted changes, not raw commit messages |
| `technical_highlights` | 1-2 | Engineering decisions (`tech` mode) or outcome metrics (`report` mode) |
| `current_status` | 1 sentence | Where the project stands now |
| `next_steps` | 1-2 | What comes next |

**When commit info is insufficient:** Infer module from file paths, intent from diffs. When truly impossible to determine → mark as "待确认" (pending confirmation). Fabricating content undermines the report's credibility — the presenter will need to stand behind these words, so they need real information they can verify.

### Step 3: Storyline

Each project follows a narrative structure. The mode determines depth:

**`tech` mode (6-part, default):**
1. Goal (Why) → What the project was trying to achieve
2. Problems (Pain) → What was blocking progress
3. Key Changes (What) → Abstracted changes
4. Technical Approach (How) → Architecture decisions, design choices
5. Result (Impact) → Measurable outcomes
6. Risk & Next Steps → Open issues and planned actions

**`report` mode (condensed to 4-part):**
1. Goal (Why) → Same
2. Key Changes → Merged What + How, higher-level
3. Result (Impact) → Same
4. Next Steps → Same

**Sparse data** (0-2 meaningful commits): Combine into a single "Status Update" slide. A slow week doesn't need 4 inflated slides.

### Step 4: PPT Stitching

Per-project slide budget by priority:

| Priority | Slides | Depth |
|----------|--------|-------|
| Core (1st) | 3-4 | Full narrative |
| Supporting (2nd) | 2-3 | Focused on key changes |
| Exploratory (3rd+) | 1-2 | Status update only |

**Overview slide:** 1 sentence per project. Natural cross-project themes are fine here when they genuinely emerge (e.g., "both projects improved reliability this week"), but don't force connections that don't exist.

**Final slide:** Aggregate next steps from all projects, one section per project.

## Output Format

Use this template:

```markdown
---
# Slide 1: 标题页
## 本周工作汇报
**日期：** YYYY-MM-DD | **范围：** project-a / project-b / project-c

---
# Slide 2: 本周总览
- **project-a：** <one-sentence summary>
- **project-b：** <one-sentence summary>
- **project-c：** <one-sentence summary>

---
# Slide 3: project-a — 背景 & 问题
## 背景
<weekly_goal>
## 本周核心问题
<core_problems>

---
# Slide 4: project-a — 关键改动 & 技术方案
## 关键改动
<key_changes>
## 技术方案
<technical_approach>

---
# Slide 5: project-a — 成果 & 下一步
## 当前成果
<current_result>
## 风险 & 下一步
<risks_and_next_steps>

---
# Final: 总结 & 下一步
## 全局总结
| 项目 | 状态 | 关键进展 |
|------|------|----------|
| project-a | <status> | <summary> |
| project-b | <status> | <summary> |

## 各项目下一步
- **project-a：** <next_steps>
- **project-b：** <next_steps>
```

## Anti-Patterns

- **commit流水账** — Listing commits verbatim ("修复bug", "调整逻辑", "优化代码"). This happens when Step 2 is skipped. Always group and abstract into engineering-level descriptions.

- **项目拼接** — Each project gets a different slide format. Apply the same narrative template uniformly — vary depth by priority, not structure.

- **没有目标** — Describing "what happened" without "why it matters." Every project needs a `weekly_goal` that frames the work. "做了一些优化" is not a goal.

- **跨项目强行融合** — Inventing cross-project themes in project-specific slides. Natural connections are welcome on the overview slide, but each project's narrative stays independent.

- **Raw git appendix** — Logs are input for analysis, not output for presentation. If the audience needs commit details, link to the repo.
