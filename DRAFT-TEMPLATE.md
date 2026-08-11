# <task-id>
<!-- kebab-case, ends in -2026-001; e.g. vendor-dispute-2026-001.
     Folder name must match. Delete all comments before submitting. -->

## Target
<!-- 2-4 sentences: which entities/datasets this is built on (use real ids from
     the explorer), what makes it hard, and which wrong paths exist. -->

## Prompt (agent-facing — underspecified)
<!-- Exactly what the agent will be told. A realistic request from a person at
     the company. NO table/file/tool/column names. May anchor a date.
     One paragraph is ideal. -->

## Servers to attach
<!-- Which surfaces the agent needs, by capability, e.g.:
     "file access, the mail tool, the expenses tool" + 1-2 decoy surfaces.
     The owner maps these to the harness and finalizes. -->

## Expected answer (with evidence chain)
<!-- The complete correct answer. Then, for EVERY fact in it, one line:
       fact -> where it comes from -> which query in queries.sql proves it
     Example:
       approver of R2026-0041 is E9003 -> per-claim lookup -> Q3 -->

## Proposed rubrics
<!-- Numbered list. Atomic: "The [variable] is [value]." Pass/fail only.
     No negatives, no and/or, no explanations.
     Sets: one count rubric + one membership rubric per member.
     15-35 total. Mark which rubrics a given wrong path would break. -->
1.
2.
3.

## Traps exploited
<!-- Which existing imperfections the wrong paths run into (name the exact
     rows/ids), and any NEW data you propose (spec it precisely: table,
     columns, rows, and what story it tells — the owner generates it;
     proposed data must be truthful in-world and additive-only). -->

## Sketch of the ideal solution path
<!-- Ordered outline of the tool calls a perfect agent would make
     (discovery -> per-item checks -> cross-checks -> answer).
     Target: 30+ calls. You don't execute this; the owner captures it. -->

## Author
<!-- Your name/handle, so the accepted task credits you. -->
