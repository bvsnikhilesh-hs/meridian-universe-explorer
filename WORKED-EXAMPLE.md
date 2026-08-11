# Worked example — a complete draft, annotated

> **Do not submit this task** — it is the shared teaching example and the owner
> already knows it. Everything below was built purely by querying the explorer.

How it was found: browsing `pto_2026` next to `oncall`, one join question is
irresistible — *does anyone's approved time off overlap their own on-call
week?* The query (Q1 below) returns exactly two people, and their stories
differ in exactly the way a good task needs: one arranged cover (provable only
in the mailbox), one did not.

---

# oncall-pto-conflict-2026-001

## Target
`oncall` × `pto_2026` overlap yields exactly two conflicted employees: E1291
(Sven Park, Storefront, week 2026-08-17) and E1552 (Support, week 2026-12-21).
E1291 arranged a swap — but only by email (thread TH-07), and the published
rota was deliberately never updated; the approving manager's reply says the
thread is the source of truth. E1552 arranged nothing. Wrong paths: trusting
the rota file as final (misses the swap), counting only one conflict (stopping
at the first hit), and a name hazard — six employees are named "Sven Park", so
name-based reporting is ambiguous; ids are required.

## Prompt (agent-facing — underspecified)
"HR asked us to sanity-check coverage for the rest of the year: is anyone
scheduled to be on call during a week they also have approved time off? For
each person you find, tell me who they are, which team and week is affected,
and whether coverage has actually been sorted out — and if it has, who is
actually taking that week."

## Servers to attach
File access, the mail tool; decoys: the note-taking and reasoning scratch
tools. (No analytics tools needed.)

## Expected answer (with evidence chain)
Two employees have approved PTO overlapping their own on-call week:
1) **E1291 (Sven Park, Storefront, week of 2026-08-17)** — PTO 2026-08-17 to
   2026-08-21 (Q1). Coverage IS sorted: an approved swap exists in the
   mailbox; **E1317 (Hiro Adebayo)** takes 2026-08-17 and E1291 takes E1317's
   week of 2026-08-24 in exchange; manager Greg Olsen (E7) approved and stated
   the published schedule would not be re-issued (Q3, Q4).
2) **E1552 (Ivan Mueller, Support, week of 2026-12-21)** — PTO 2026-12-18 to 2026-12-22
   (Q1). No swap, no cover arrangement exists anywhere in the mailbox (Q5).
Evidence chain:
- the two conflicts → oncall×pto join → Q1
- E1291/E1552 identities & teams → roster lookup → Q2
- swap exists, partner E1317, manager approval, rota deliberately stale → mail
  thread TH-07 bodies → Q3, Q4
- E1552 has no cover → mailbox sweep for that id/week → Q5
- "Sven Park" is 6-way ambiguous (ids mandatory) → Q6

## Proposed rubrics
1. The number of employees whose approved time off overlaps their own on-call
   week is 2.
2. E1291 is one of the employees with an overlapping on-call week.
3. E1552 is one of the employees with an overlapping on-call week.
4. E1291's affected on-call week is the Storefront week starting 2026-08-17.
5. E1552's affected on-call week is the Support week starting 2026-12-21.
6. The employee actually covering Storefront's week of 2026-08-17 is E1317.
7. The manager who approved the swap for the week of 2026-08-17 is Greg Olsen (E7).
8. The week E1291 takes in exchange is the Storefront week starting 2026-08-24.
9. The published on-call schedule still lists E1291 for the week of 2026-08-17.
10. The number of cover arrangements found for E1552's week is 0.
11. The source establishing the swap is the company mailbox.
    <!-- wrong-path pricing: trusting the rota alone breaks 6,7,8,9,11;
         stopping at one conflict breaks 1 and 3/5/10;
         name-only reporting breaks 2/3 (ids absent). -->

## Traps exploited
The deliberately stale rota vs. the mail thread (TH-07 states the file will not
be re-issued); the 6-way "Sven Park" name collision (Q6) — including a
same-team collision inside Storefront; decoy swap threads in the mailbox that
were declined (TH-22) or never answered (TH-23) and must not be reported as
cover; other PTO rows for other Sven Parks in non-conflicting weeks. No new
data needed.

## Sketch of the ideal solution path
List the sandbox → read the rota and PTO files → compute overlaps (2 hits) →
roster lookups for both ids → mailbox: search on-call-related labels/terms →
read every message in the three swap-ish threads (approved / declined /
unanswered — 8 bodies) → confirm the approved one covers exactly the affected
week and extract partner + approver → sweep the mailbox for E1552 / December
cover (negative result established positively by exhaustive thread listing) →
verify the rota file still shows E1291 (staleness) → answer. (~30–34 calls.)

## Author
(worked example — benchmark owner)

---

## And the matching `queries.sql`

```sql
-- Q1: PTO overlapping the employee's OWN on-call week (the anchor finding)
SELECT p.emp_id, o.team, o.week_start, p.start_date, p.end_date
FROM pto_2026 p
JOIN oncall o ON o.emp_id = p.emp_id
WHERE date(o.week_start) <= date(p.end_date)
  AND date(o.week_start, '+4 days') >= date(p.start_date);

-- Q2: identities (ids are mandatory — see Q6)
SELECT emp_id, name, team FROM employees WHERE emp_id IN ('E1291','E1552');

-- Q3: the swap thread exists and names the week
SELECT id, "from", date, subject FROM mails
WHERE thread_id = 'TH-07' ORDER BY date;

-- Q4: the bodies prove partner, approval, and the deliberately stale rota
SELECT id, body FROM mails WHERE thread_id = 'TH-07' ORDER BY date;

-- Q5: no cover exists for E1552 / the December week (positive exhaustive check)
SELECT COUNT(*) AS mentions FROM mails
WHERE body LIKE '%E1552%' OR body LIKE '%2026-12-21%' OR body LIKE '%2026-12-18%';

-- Q6: the name hazard — "Sven Park" is 6-way ambiguous
SELECT emp_id, team FROM employees WHERE name = 'Sven Park';
```

**Why this draft would be accepted:** the anchor is a genuine data fact (Q1),
every rubric fact has a query, the wrong paths each break ≥3 rubrics, the
prompt names no tools or files, rubrics are atomic with a count+membership set
and zero negatives, and the ideal path needs the mail tool's per-message reads
— it can't be shortcut by one file read.
