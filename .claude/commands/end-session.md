# End CLIF Session

Wrap up the current session and persist state for next time.

## 1. Session Summary
- What was the **objective** for this session?
- What was **accomplished** at a high level?

## 2. Work Completed
Itemize specific outputs:
- Files created or modified
- Analyses run and notebooks executed (marimo/Quarto/RMarkdown)
- CLIF tables used or generated
- Data outputs produced (CSV, Parquet, plots, intermediate files)
- Cohort sizes at each inclusion/exclusion step
- Decisions finalized

## 3. Decisions Made
For each significant decision:
- What was decided
- Why (rationale, evidence, clinical reasoning)
- What alternatives were considered

## 4. Open Items / Next Steps
- Remaining tasks or analyses
- Blockers or dependencies
- Pending questions (statistical, clinical, data quality)
- Data quality observations or concerns

## 5. Notes
- Assumptions made during this session
- Technical debt or cleanup needed
- Site-specific observations
- Any contextual details that future sessions need

## 6. State Persistence Protocol

Update the following files to ensure continuity:

### `./.claude/claude-progress.md`
Append a new session entry:
```
## Session — YYYY-MM-DD
**Summary:** [1-2 sentence overview]
**Work completed:**
- [item 1]
- [item 2]
**CLIF tables / data notes:** [tables used, data quality observations, cohort sizes]
**Decisions:** [key choices and rationale]
**Open items:** [what remains]
**Notes:** [anything else]
```

### `./.claude/claude-todo.md`
- Mark completed items as done
- Add any newly identified tasks
- Re-prioritize if needed

### `./.claude/lessons.md`
- Document any corrections received
- Record patterns or anti-patterns discovered
- Note any reusable approaches or pitfalls
