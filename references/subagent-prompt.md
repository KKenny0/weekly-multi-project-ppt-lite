# Subagent Prompt Template

Fill in `{placeholders}` and send to each subagent. Dispatch multiple projects in parallel.

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

If 0 change_blocks after filtering → "maintenance week". Output empty results and note it.

**Step 2: Abstract into engineering semantics**
Group related commits into abstracted changes. Example: "add auto-layout" + "fix overlap" + "add positioning" → ONE key_change: "Built scene composition system with auto-layout and dense-panel overlap resolution"

Constraints: core_problems ≤ 2, key_changes ≤ 4.

When commit info is insufficient: infer from file paths or diffs. When truly impossible → mark as "待确认". Do NOT fabricate.

**Step 3: Build narrative**

Tech mode (6-part): Goal(Why) → Problems(Pain) → KeyChanges(What) → TechApproach(How) → Result(Impact) → Risk&Next

Report mode (4-part): Goal(Why) → KeyChanges → Result(Impact) → NextSteps

Sparse data (0-2 commits): combine into a single "Status Update".

**Return ONLY this JSON (no other commentary):**

{
  "project": "{project_name}",
  "priority": "{priority}",
  "is_maintenance_week": false,
  "narrative": {
    "goal": "1 sentence — why this week's work matters",
    "problems": "core pain points addressed (≤ 2)",
    "key_changes": "abstracted engineering changes (≤ 4)",
    "technical_approach": "how it was done (omit in report mode)",
    "result": "impact and outcomes",
    "risk_and_next": "risks and next steps"
  }
}
```

## Handling Results

- `is_maintenance_week: true` → project gets only a brief line on the overview slide, no dedicated project slides
- Non-JSON response → retry with stricter format instruction
- Missing fields → fill from available data; if truly missing → mark as "待确认"
