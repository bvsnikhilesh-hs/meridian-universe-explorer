# Tool Surfaces — what the agent under test can actually call

Auto-generated from the live server schemas on 2026-08-11. **Your solution-path
sketch must only reference tools on this page, with these exact parameters.**
If a capability isn't listed, the agent doesn't have it — design accordingly.

## `linear` — Ticketing view (issues <- support tickets; users <- employees WITHOUT manager info; customers <- customer records). No contact registry here — contacts are a file.

- **list_issues**(assignee, createdAt, cursor, cycle, delegate, includeArchived, label, limit, orderBy, parentId, priority, project, query, release, state, team, updatedAt)
- **get_issue**(id*, includeCustomerNeeds, includeRelations, includeReleases)
- **list_teams**(createdAt, cursor, includeArchived, limit, orderBy, query, updatedAt)
- **get_team**(query*)
- **list_users**(cursor, limit, orderBy, query, team)
- **get_user**(query*)
- **list_projects**(createdAt, cursor, includeArchived, includeMembers, includeMilestones, initiative, label, limit, member, orderBy, query, state, team, updatedAt)
- **get_project**(includeMembers, includeMilestones, includeResources, query*)
- **list_issue_statuses**(team*)
- **list_issue_labels**(cursor, limit, name, orderBy, team)
- **list_cycles**(teamId*, type)
- **list_comments**(cursor, documentId, initiativeId, issueId, limit, milestoneId, orderBy, projectId, statusUpdateId, statusUpdateType)
- **list_customers**(createdAt, cursor, includeArchived, includeNeeds, limit, orderBy, owner, query, status, tier, updatedAt)
- **search_documentation**(page, query*)

## `mail` — Company mailbox. Search returns summaries only (id, thread, from, date, subject, labels); reading a body is one call per message.

- **search_emails**(maxResults, query)
- **read_email**(messageId*)
- **list_email_labels**((no args))

## `expenses` — Reimbursement claims. The list view HIDES the approver — per-claim lookups reveal it. Users carry no manager field.

- **list_reimbursements**(limit, status, user_id)
- **get_reimbursement**(reimbursement_id*)
- **list_users**(department)
- **get_user**(user_id*)
- **list_departments**((no args))

## `airtable` — Marketing-ops base (its OWN datasets — not the company files).

- **list_bases**((no args))
- **list_tables**(baseId*, detailLevel)
- **describe_table**(baseId*, detailLevel, tableId*)
- **list_records**(baseId*, fields, filterByFormula, maxRecords, sort, tableId*, view)
- **search_records**(baseId*, fieldIds, maxRecords, searchTerm*, tableId*)
- **get_record**(baseId*, recordId*, tableId*)

## `mongodb` — Commerce store `meridian_commerce` (its OWN datasets — not the company files).

- **list-databases**((no args))
- **list-collections**(database*)
- **collection-schema**(collection*, database*)
- **find**(collection*, database*, filter, limit, projection, sort)
- **aggregate**(collection*, database*, pipeline*)
- **count**(collection*, database*, query)
- **collection-indexes**(collection*, database*)
- **explain**(collection*, database*, filter, method, pipeline)

## `filesystem` — every file in the world folder (CSV/JSON/TXT)

- **list_directory**(path*), **directory_tree**(path*), **search_files**(path*, pattern*)
- **read_text_file**(path*, head, tail), **read_multiple_files**(paths*)
- **get_file_info**(path*) — plus write/edit tools (writes only affect the agent's sandbox copy)
- NOTE: long files are returned truncated (~8k chars per read); `head`/`tail` help

## Utility & decoy servers

- **calculator**: calculate(expression*) — exact arithmetic
- **time**: get_current_time, convert_time — timezone/date math
- **git** / **code-runner** — code-domain only (repo tasks)
- **memory**, **everything**, **sequentialthinking** — decoy surfaces; hold no world data

## What is reachable where (quick matrix)

| Data | filesystem | linear | mail | expenses | airtable | mongodb |
|---|---|---|---|---|---|---|
| employees / org chart | YES (only place managers exist) | users (no manager) | — | users (no manager) | — | — |
| tickets | YES | YES (as issues) | — | — | — | — |
| customers | YES | YES | — | — | — | — |
| customer CONTACTS | YES (registry file) | NO | addresses appear in mail | — | — | — |
| orders / refunds / invoices / PTO / HR / policies | YES | — | — | — | — | — |
| expense claims | YES (file) | — | — | YES (approver only per-claim) | — | — |
| mailbox | YES (raw file) | — | YES (search + per-message read) | — | — | — |
| marketing metrics (airtable_*) | NO | — | — | — | YES only | — |
| commerce store (mongo_*) | NO | — | — | — | — | YES only |

(* = required parameter. Explorer table names `airtable_*`/`mongo_*` map to
those two tools' tables/collections; everything else in the explorer is a file.)
