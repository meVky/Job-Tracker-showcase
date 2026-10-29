# MCP tools

The Job Tracker MCP server lets an AI assistant such as Claude Desktop or Claude Code read and add to a user's job pipeline. It exposes three tools, deliberately few and safe.

## Transports

- **stdio**, for a single user running it locally with their own configuration.
- **Streamable HTTP with OAuth 2.1**, for connecting to a hosted instance. Each token belongs to one user, and tools act only on that user's data.

## Design principles

- **Reuse the app's logic.** Tools call the same service functions as the web app, so an agent can never see numbers that differ from the Analytics page.
- **Read tools are read-only.** They cannot change anything.
- **Writes are two-step.** A write tool previews first and only writes when called a second time with an explicit confirmation.
- **Small surface.** Three tools cover the common questions. More can be added on the same pattern.

## Tools

### `list_jobs`
Lists jobs, most recently updated first. Read-only.

| Parameter | Type | Default | Notes |
|---|---|---|---|
| `status` | string, optional | none | One of `new`, `saved`, `applied`, `interviewing`, `offer`, `rejected` |
| `limit` | integer | 20 | Maximum number of jobs returned |

Returns one line per job: `[id] Role @ Company, status: applied`.

### `get_pipeline_summary`
Summary statistics for the search. Read-only.

| Parameter | Type | Default | Notes |
|---|---|---|---|
| `contract_type` | string, optional | none | For example `Permanent`, `Internship` |

Returns: total jobs, applied, currently interviewing, offers, rejected, active (in progress), response rate, and the average days to interview and to offer when they can be computed. It calls the same function as the Analytics page, so the numbers always match the app.

### `add_job_manual`
Adds a job. **Writes data**, so it is deliberately two-step.

| Parameter | Type | Default | Notes |
|---|---|---|---|
| `company` | string | required | |
| `role` | string | required | |
| `url` | string, optional | none | |
| `status` | string | `new` | Same values as `list_jobs` |
| `contract_type` | string, optional | none | "Full-time" is accepted and stored as `Permanent`; also `Part-time`, `Contract`, `Freelance`, `Internship` (case-insensitive) |
| `location`, `salary_range`, `seniority`, `notes` | string, optional | none | |
| `due_date` | ISO date, optional | none | For example `2026-09-20` |
| `confirm` | boolean | `false` | Must be `true` to write |

How the two steps work:

1. Call with `confirm=false`. Nothing is created. The reply is a preview of the job and a warning if a job with the same company and role already exists.
2. After the user has seen the preview, call again with `confirm=true` to create it.

Because a write needs a second, explicit call, a single misread model turn cannot change the database.

## Example conversation

> **You:** What does my pipeline look like?
> **Claude:** *(calls `get_pipeline_summary`)* You are tracking 42 jobs, 3 interviewing, response rate 38%.
>
> **You:** Add a Product Manager role at ExampleCorp, applied today.
> **Claude:** *(calls `add_job_manual` with `confirm=false`)* This would create: ExampleCorp, Product Manager, status applied. No duplicate found. Shall I add it?
> **You:** Yes.
> **Claude:** *(calls `add_job_manual` with `confirm=true`)* Done.
