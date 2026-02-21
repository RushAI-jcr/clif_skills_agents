# CLIF Project Workflow Rules

## Language Detection
Detect project language from file extensions and config files:
- **R project**: `.R`, `.Rmd`, `.qmd`, `renv.lock`, `config/config_template.yaml`
- **Python project**: `.py`, `pyproject.toml`, `uv.lock`, `clif_config.json`
- If ambiguous, ask the user

Use language-appropriate skill references and code examples throughout.
Only load and reference skills matching the detected project language. Check each skill's `language` frontmatter field (`python`, `r`, or `both`). Do NOT suggest or reference skills from the other language's stack.

## CLIF Data Rules (MANDATORY)

- **Always filter on `_category` columns, never `_name` columns.** Names are site-specific EHR strings. Categories are the CLIF controlled vocabulary. This is the #1 source of cross-site bugs.
- **Always validate data before analysis.** Python: `clif.validate_all()` / R: verify schemas against TABLES.md or use CLIF-TableOne validation.
- **Use ADT table for ICU times, not hospitalization table.** `hospitalization.admission_dttm` is hospital admission, not ICU admission. Filter `adt` where `location_category == 'icu'`.
- **Log row counts at every cohort filter step.** CONSORT-style attrition tracking is required.
- **Use encounter stitching for ICU stay identification.** A single ICU stay may span multiple hospitalization records.
- **Distinguish continuous vs intermittent medications.** `medication_admin_continuous` = drips/infusions. `medication_admin_intermittent` = boluses/scheduled.
- **All timestamps are UTC.** Timezone conversion is mandatory — Python: clifpy's `timezone` parameter / R: `lubridate::with_tz()`.
- **When creating figures, follow JAMA style:** Arial font, minimum 8pt text, no overlap, legends outside plot area, sentence case axis labels.

## Documentation Lookup (MANDATORY)

- **Always check [clif-icu.com](https://clif-icu.com) for current CLIF code examples, schemas, and API usage before writing CLIF-related code.** This is the authoritative source — local skill references may be outdated.
- **Before writing or updating code that uses a library** (clifpy, Polars, arrow, marimo, etc.), use Context7 MCP (`resolve-library-id` → `query-docs`) or Ref MCP (`ref_search_documentation` → `ref_read_url`) to fetch the latest documentation. Do not rely on training data for API signatures or function parameters.
- **When code examples in skills conflict with live docs**, the live docs win. Update the code to match, and flag the discrepancy to the user.
- If the user asks you to update coding examples in the skills, do so — but always verify against clif-icu.com and the relevant library docs first.

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately — don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user, update `.claude/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Iterate on these lessons until mistake rate drops
- Review lessons at session start for the relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Run code, check outputs, validate data shapes and counts
- Ask yourself: "Would a senior researcher approve this?"
- For analyses: verify sample sizes, check for NaN/missing data, confirm statistical assumptions

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes — don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests — then resolve them
- Zero context switching required from the user

---

## Task Management

1. **Plan First**: Write plan to `.claude/claude-todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `.claude/claude-todo.md`
6. **Capture Lessons**: Update `.claude/lessons.md` after corrections

---

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.
- **Reproducibility**: All analysis must be reproducible. Config-driven, version-controlled, documented.
