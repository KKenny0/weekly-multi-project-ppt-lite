# weekly-multi-project-ppt-lite

A Claude Code skill that generates structured Markdown PPT outlines from multiple projects' weekly git commit history.

> **Note:** This skill has been migrated to the [work-sync-skills](https://github.com/KKenny0/work-sync-skills) monorepo.
> This repo is archived and no longer maintained.

## What It Does

Parses git logs across multiple repositories, classifies and abstracts commits through parallel subagents, then assembles a presentation-style weekly report with per-project narratives.

## Pipeline

```
Phase 0  Scope Gathering    (main dialog)    parse prompt → collect git logs
Phase 1  Per-Project Analysis (subagent × N)  classify → abstract → build narrative
Phase 2  PPT Stitching      (main dialog)    structured JSONs → Markdown outline
```

Phase 1 runs in parallel across projects for token efficiency.

## Install

This skill is now included in the [work-sync-skills](https://github.com/KKenny0/work-sync-skills) plugin. Install the plugin to get this skill along with others.

## Usage

Trigger phrases:

- "生成周报PPT"
- "多项目周报"
- "weekly report"
- Any request to consolidate multiple repos' git activity into a presentation

### Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| Time range | this week | `this_week` or `last_week` |
| Projects | (required) | Name + repo path for each project |
| Mode | tech | `tech` (6-part narrative) or `report` (4-part outcomes) |
| Priority | auto | Auto by commit count, or user-specified |

### Example

```
帮我生成本周的周报PPT，项目：
1. frontend-app D:/projects/frontend
2. backend-api D:/projects/api
```

## File Structure

```
weekly-ppt-lite/
├── SKILL.md                      # Skill definition
├── references/
│   ├── subagent-prompt.md        # Phase 1 analysis template
│   └── slide-template.md         # Phase 2 slide structures (tech + report)
└── README.md
```

## Related

- [work-sync-skills](https://github.com/KKenny0/work-sync-skills) — The monorepo that now contains this skill
- [weekly-report-ppt](https://github.com/KKenny0/weekly-report-ppt) — For actual `.pptx` file generation (this skill outputs Markdown outlines only)
