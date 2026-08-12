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

## `assets` — IT asset inventory (Snipe-IT-style; its OWN dataset `asset_inventory.json`). Search covers tag/serial/model/notes — NOT assignees.
- `list_assets(status, assigned_to, model, location, limit)` — List assets, optionally filtered. Filters (status, assigned_to, model, location) are ANDed. status is one of: deployed, loaner, in-repair, r
- `get_asset(tag)` — Retrieve the full record of a specific asset by tag: tag, serial, model, status, assigned_to, location, checked_out, and notes (when present
- `search_assets(query)` — Search assets. Query grammar: space-separated terms, all of which must match (AND), case-insensitively, against tag, serial, model NAME, and
- `list_models()` — List the hardware model registry: id, name, category, and per-model asset count.
- `list_locations()` — List the location REGISTRY: registered location names with per-location asset counts. Note: this is the registry as maintained, not a roll-u

## `calendar` — Company calendar (Google-Calendar-style; its OWN dataset `calendar_2026.json`). Search covers title+location only; recurring series stored unexpanded.
- `list_calendars()` — List all calendars: id, name, and per-calendar event count.
- `list_events(calendarId, timeMin, timeMax, maxResults)` — List events on one calendar, optionally restricted to a time window. Window semantics follow the Google Calendar API: an event matches when 
- `search_events(query, maxResults)` — Search events across ALL calendars. Query grammar: space-separated terms, all of which must match (AND), case-insensitively, against title +
- `get_event(eventId)` — Retrieve the full record of a specific event by ID: id, calendarId, title, start, end, organizer, attendees, location, status, recurrence.

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
