---
name: weekly-multi-project-ppt-lite
description: Use when generating weekly PPT outlines from multiple projects' git commit history, when user says "生成周报PPT", "多项目周报", "weekly report", "周报", or when asked to summarize git history across projects into presentation format
---

# Weekly Multi-Project PPT Lite

## Overview

Convert **multiple projects' weekly git commits** into a structured Markdown PPT outline. Each project gets an independent narrative using a unified template. The overview slide can note natural cross-project themes, but each project's slides stay self-contained.

## When to Use

- Multiple projects running in parallel
- User provides git commit history and wants a presentation-style report
- Projects have independent goals

**Not for:** Single-project reports, actual .pptx generation (use weekly-report-ppt), non-git-based reporting.

## Inputs

| Parameter | Required | Default | How to resolve |
|-----------|----------|---------|----------------|
| Time range | Optional | `this_week` | "本周" → this_week; "上周" → last_week |
| Projects (name + repo path) | Yes | — | From prompt, or ask user |
| Mode: `tech` / `report` | Optional | `tech` | Infer from context |
| Priority order | Optional | Auto | Infer from commit volume |

**Mode controls output depth:**
- `tech` — Full 6-part narrative per project, includes architecture decisions and design choices
- `report` — Condensed 4-part (Goal → Changes → Impact → Next), focuses on outcomes over implementation

## Pipeline

```
Phase 0 (main dialog):
  parse prompt → confirm scope → collect git logs

Phase 1 (subagent, per project, parallelizable):
  git logs → classify → abstract → storyline → structured JSON

Phase 2 (main dialog):
  structured JSONs → PPT stitching → final Markdown outline
```

Phase 0 gathers scope and collects raw git data. Phase 1 does the expensive per-project analysis in subagents (classifying commits, abstracting, building narrative). Phase 2 assembles the compact summaries into the final PPT outline.

## Phase 0: Scope Gathering

Parse the user's prompt and collect missing parameters before dispatching subagents.

### Parameter Resolution

- **Time range**: Parse from prompt ("本周"/"上周", "this week"/"last week"). Default: `this_week`. Only ask if ambiguous.
- **Projects**: Parse project names and repo paths from prompt. If missing → ask: "请提供要包含的项目名称和 repo 路径". This is the only parameter that cannot be defaulted.
- **Mode**: Infer from context. Default: `tech`.

Don't confirm defaults — proceed when all required parameters are resolved.

### Date Calculation

- `this_week`: current week's Monday → today
- `last_week`: previous week's Monday → previous week's Sunday

### Git Log Collection

For each project, run:
```bash
git -C <repo_path> log --oneline --since="<start_date>"
```

For `last_week`, also add `--until="<this_monday_date>"`. Dates are YYYY-MM-DD format.

If a repo has no commits in the range, note it as a "maintenance week" for that project.

Pass the collected logs into Phase 1's subagent prompt template as `{git_logs}`.

## Phase 1: Dispatch Subagents

For each project, dispatch a subagent with the prompt template below. Multiple projects can be dispatched in parallel.

### Subagent Prompt Template

Fill in the `{placeholders}` and send to each subagent:

```text
You are a weekly report analyst. Analyze the following project's git commits for this week's report.

Project: {project_name} (priority: {priority})
Mode: {tech_or_report}

Git logs:
{git_logs}

Execute these 3 steps in order:

**Step 1: Classify commits**
- Keep: feat/feature, fix, refactor, perf
- Drop: chore, docs, style
- Output: change_blocks list with { type, module, summary }

If 0 change_blocks after filtering → this is a "maintenance week". Output empty results and note it.

**Step 2: Abstract into engineering semantics**
Group related commits into abstracted changes. Example: "add auto-layout" + "fix overlap" + "add positioning" → ONE key_change: "Built scene composition system with auto-layout and dense-panel overlap resolution"

Constraints: core_problems ≤ 2, key_changes ≤ 4.

When commit info is insufficient: infer from file paths or diffs. When truly impossible → mark as "待确认". Do NOT fabricate — the presenter needs real information they can verify.

**Step 3: Build narrative**

Tech mode (6-part): Goal(Why) → Problems(Pain) → KeyChanges(What) → TechApproach(How) → Result(Impact) → Risk&Next

Report mode (4-part): Goal(Why) → KeyChanges → Result(Impact) → NextSteps

Sparse data (0-2 commits): combine into a single "Status Update".

**Return ONLY this JSON (no other commentary):**

{
  "project": "{project_name}",
  "priority": "{priority}",
  "weekly_goal": "1 sentence",
  "core_problems": ["≤ 2 items, or empty if maintenance week"],
  "key_changes": ["≤ 4 abstracted changes, or empty"],
  "technical_highlights": ["1-2 items"],
  "current_status": "1 sentence",
  "next_steps": ["1-2 items"],
  "narrative": {
    "goal": "...",
    "problems": "...",
    "key_changes": "...",
    "technical_approach": "... (omit in report mode)",
    "result": "...",
    "risk_and_next": "..."
  },
  "is_maintenance_week": false
}
```

### Handling Results

Collect the JSON from each subagent. If a subagent returns `is_maintenance_week: true`, that project gets only a brief line on the overview slide — no dedicated project slides.

## Phase 2: PPT Stitching (Main Dialog)

Using the collected JSONs, assemble the final Markdown PPT outline.

### Slide Structure

```markdown
---
# Slide 1: 标题页
## 本周工作汇报
**日期：** YYYY-MM-DD | **范围：** project-a / project-b / project-c

---
# Slide 2: 本周总览
- **project-a：** {weekly_goal}
- **project-b：** {weekly_goal}
- **project-c：** {weekly_goal}

---
# Slide 3: project-a — 背景 & 问题
## 背景
{narrative.goal}
## 本周核心问题
{narrative.problems}

---
# Slide 4: project-a — 关键改动 & 技术方案
## 关键改动
{narrative.key_changes}
## 技术方案
{narrative.technical_approach}

---
# Slide 5: project-a — 成果 & 下一步
## 当前成果
{narrative.result}
## 风险 & 下一步
{narrative.risk_and_next}

---
# Final: 总结 & 下一步
## 全局总结
| 项目 | 状态 | 关键进展 |
|------|------|----------|
| project-a | {current_status} | {weekly_goal} |
| project-b | {current_status} | {weekly_goal} |

## 各项目下一步
- **project-a：** {next_steps}
- **project-b：** {next_steps}
```

### Slide Budget by Priority

| Priority | Slides | Depth |
|----------|--------|-------|
| Core (1st) | 3-4 | Full narrative |
| Supporting (2nd) | 2-3 | Focused on key changes |
| Exploratory (3rd+) | 1-2 | Status update only |

### Overview Slide

1 sentence per project. Natural cross-project themes are fine here when they genuinely emerge (e.g., "both projects improved reliability this week"), but don't force connections that don't exist.

### Final Slide

Aggregate next steps from all projects, one section per project.

## Anti-Patterns

- **commit流水账** — Listing commits verbatim. This happens when abstraction is skipped. Always group and abstract into engineering-level descriptions.

- **项目拼接** — Each project gets a different slide format. Apply the same narrative template uniformly — vary depth by priority, not structure.

- **没有目标** — "what happened" without "why it matters." Every project needs a `weekly_goal`. "做了一些优化" is not a goal.

- **跨项目强行融合** — Inventing cross-project themes in project-specific slides. Natural connections are welcome on the overview slide only.

- **Raw git appendix** — Logs are input for analysis, not output for presentation. If the audience needs commit details, link to the repo.
