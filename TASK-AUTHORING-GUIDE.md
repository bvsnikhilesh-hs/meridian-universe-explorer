# Meridian Benchmark — Task Authoring Guide for Fellows

You are drafting evaluation tasks for an MCP-Atlas-style benchmark. An AI agent
gets dropped into a synthetic company ("Meridian") that it can only see through
tools, receives a vague-but-realistic request, and is graded on whether its
final answer contains specific verifiable facts.

**Your job ends at the draft.** You design the task and prove its answer with
SQL. The benchmark owner does everything else: verifier code, golden runs,
difficulty calibration (pass@8 across four frontier models), thresholds, and
acceptance. You never run validation.

---

## 1. The explorer

**https://bvsnikhilesh-hs.github.io/meridian-universe-explorer/**

Works in any browser, nothing to install. The sidebar lists all 31 tables with
row counts — click one to see its columns and load a preview query. Write any
SQL (SQLite dialect); it runs locally in your tab. `Ctrl/Cmd+Enter` runs.

Practical notes:
- In `mails`, the `to`, `cc`, and `labels` columns are `;`-joined lists — use
  `LIKE '%…%'`.
- In `mongo_*` tables, nested objects are stored as JSON text — use
  `json_extract(col, '$.field')`.
- Quote column names that collide with keywords: `"from"`, `"to"`.
- Your writes (temp views etc.) only exist in your tab. Refresh = reset.

## 2. The world in one page

Three original domains plus newer verticals, all deliberately interlinked
**and deliberately imperfect** — the imperfections are the benchmark.

| Table(s) | What it is | How the agent under test reaches it |
|---|---|---|
| employees (721) | 3-level org chart; the ONLY place managers are recorded | file read |
| oncall (520) | weekly on-call rota, 10 teams × 52 weeks of 2026 | file read |
| services (64) | service → owning team registry | file read |
| customers (201), tickets (420) | accounts + support tickets | file read, and as "issues/customers" via a Linear-style tool |
| orders_2024/2025/2026, refunds | order ledgers + refunds (no refund ids/dates) | file read |
| expenses_2026 (82) | reimbursement claims | Ramp-style tool whose LIST view hides the approver — per-claim lookups required |
| invoices_2025, refund_approvals, pto_2026, hr_offboarding, customer_contacts, region_migration | finance/HR/master-data verticals | file read |
| mails (41) | company mailbox, 22 threads | Gmail-style tool: search returns summaries; bodies need per-message reads |
| documents (9) | policies, memos, advisory notes — some deliberately stale or superseded | file read |
| airtable_* (7 tables) | marketing-ops metrics base | Airtable-style tool ONLY (agents cannot read these as files) |
| mongo_* (5 tables) | commerce store (transactions, shipments, reviews…) | MongoDB-style tool ONLY |

Known trap families (verify with SQL, then build on them): duplicate rows
(orders, expenses, approvals, invoices), ids that resolve nowhere (customers on
orders, assignees on tickets, approvers on payouts), same-name people (three
"Maria Chen"s; six "Sven Park"s) and near-twin customers ("Maria Chenn"),
near-miss service names (`payments` vs `payments-api`), stale documents that
give confidently wrong routing, status labels that lie (a claim can be blocked
while merely "pending"; a ticket can be "open" with a resolved date), records
that exist only through dangling references, and week-boundary date rules
(Sunday belongs to the prior Monday's rota week — see the support playbook).

## 3. Designing a task

1. **Start from a question a real employee would ask.** "Why hasn't Luca's
   expense been paid, and who owns it now?" — not "join expenses to employees."
2. **Make the answer discrete facts**: ids, names, amounts, dates, counts.
   Never large unaided arithmetic; never opinions or summaries.
3. **Require a chain**: the answer should force 2–4 datasets to be combined,
   ideally across different tool surfaces (a file + the mail tool + a per-item
   lookup tool).
4. **Give every wrong path a landing spot.** The best tasks have a
   plausible-but-wrong answer an agent reaches by trusting a stale document,
   matching a name instead of an id, reading a status label literally, or
   stopping one hop early. If the lazy path and the correct path give the same
   answer, the task measures nothing.
5. **Multi-part answers win**: missing any part fails. Two branches from one
   anchor (an ops branch and an account branch) is a proven pattern.
6. **The prompt is underspecified.** No table names, no file names, no tool
   names, no column names. Set the scene, state the need, optionally anchor a
   date. Discovery is part of the test.
7. **Scope check**: the ideal solution should take roughly 30+ tool calls —
   favored by per-item lookups (each mail body, each claim's approver) and
   rule-outs of look-alikes, not by data volume.

**Four rules born from real failed submissions:**
- **Only reference real tools.** Your solution-path sketch may only use tools
  listed in TOOL-SURFACES.md, with their exact parameters. Never invent a tool,
  a parameter, or a tool output — if you find yourself writing what a tool
  "would" return, stop: prove it with SQL instead and leave execution to the
  owner.
- **Submit ONLY `DRAFT.md` + `queries.sql`.** Do not author task.json,
  golden_plan.json, golden_trajectory.json, or verify_answer.py — those are
  owner artifacts, generated against the real harness. Fabricated trajectory
  files are the #1 cause of rejected drafts.
- **No date anchors unless you've checked the horizon.** Before writing
  "as of <date>" in a prompt, run `SELECT max(<date_col>)` on every dataset
  your answer touches. If ANY relevant record is dated after your anchor, the
  task has two defensible answers and will bounce. Default: no anchor.
- **Every rubric must be elicited by your own prompt.** Read your prompt, then
  each rubric, and ask: "would a complete, direct answer to this prompt state
  this fact?" If the answer is no (the fact is merely *discoverable*), either
  extend the prompt so it asks for it, or delete the rubric. Rubrics that
  grade unasked-for facts fail every honest run.

What you may NOT do: propose edits to existing data (if your idea needs new
data, spec it in the draft and the owner will generate it); use benchmark
words (trap, decoy, rubric, benchmark…) in any agent-visible text; build on
facts that only you can compute — every fact must be provable by a query you
include.

## 4. Rubric rules (strict)

Rubrics are the grading contract. A judge reads the agent's final answer and
marks each rubric **pass or fail — nothing in between**.

1. One rubric = **one verifiable fact**. Form: *"The [variable] is [value]."*
   - "The amount of claim R2026-0041 is 1760."
   - "The only unresolved ticket assigned to E9001 is T5417."
2. **No explanations** inside a rubric — no "because…", "rather than…",
   "which shows…".
3. **Never join two facts** with and/or. Split them.
4. **Sets = one count rubric + one membership rubric per member.**
   - "The number of claims awaiting the departed approver is 3."
   - "R2026-0043 is one of the claims awaiting the departed approver." (×3)
5. **Negative rubrics are NOT allowed** — never "does not / never / excludes".
   Exclusion is enforced positively: if the true count is 3 and an agent lists
   a 4th item, the count rubric fails.
6. Aim for **15–35 rubrics**. Structure them so that falling for each designed
   wrong path costs **several** rubrics at once (the count + the memberships +
   the per-fact rubrics of the missed item).

## 5. What you submit

One folder, zipped, sent to the benchmark owner **via Slack**:

```
drafts/<your-task-id>/        e.g. drafts/vendor-dispute-2026-001/
  DRAFT.md                    the draft (template provided — DRAFT-TEMPLATE.md)
  queries.sql                 every query that proves your expected answer,
                              in order, with a one-line comment each
```

`queries.sql` is not optional — it is how we recompute your ground truth. If a
fact in your expected answer has no query behind it, the draft comes back.

## 6. What happens after you submit

The owner will: recompute your answer in code against the canonical data →
check it against frozen facts you can't see (expect occasional "pick a
different anchor, that one is reserved") → generate any new data you specified
→ capture a real tool-call golden run → set the pass threshold from the rubric
arithmetic → run the task 8× against four frontier models. **The bar: every
model fails at least 2 of 8 runs, and a careful agent can still pass.** Too
easy comes back for a harder chain; impossible comes back for a fairness fix.
Accepted tasks are credited to you.

Read WORKED-EXAMPLE.md and TOOL-SURFACES.md before your first draft. Then spend an hour in the
explorer before writing a word — the best tasks are found in the data, not
invented on top of it.
