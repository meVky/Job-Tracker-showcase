# Job Tracker

A full-stack job-search platform with AI matching, automatic job discovery, analytics and an MCP server so Claude can work with your pipeline. Built with Claude Code, React and FastAPI, and used by 14+ job seekers.

**Why I built it:** job hunting meant copying the same details into spreadsheets and guessing which applications were working. This tracks every stage, and shows which kinds of roles actually lead to interviews and which applications need a follow-up.

> **This repository is a showcase.** It documents the architecture, the MCP tool interface and the data model. The application source is private. All screenshots use a demo account with fictional companies; no real user data or credentials appear here.

![Job board with AI match scores and pipeline stats](docs/screenshots/Job-tracker.png)
![Career lanes: which kinds of roles lead to interviews](docs/screenshots/CareerInsights.png)

## What it does

- **Track jobs on a board.** Paste a job description or URL and AI extracts the role, company, skills and salary. Drag to reorder, and move jobs through New, Saved, Applied, Interviewing, Offer and Rejected.
- **Match against your profile.** AI compares each job with your CV and profile, with a score, hard requirements, skill gaps, cover-letter angles and likely interview questions. Every run is kept as history.
- **Discover jobs automatically.** Search through the Adzuna API, or follow companies and scan their careers pages (Greenhouse, Lever, Ashby and generic HTML). Matches go to a review queue before they become real jobs.
- **Follow-up prompts.** A pipeline pulse flags quiet applications and suggests who to chase, so nothing goes silent unnoticed.
- **Analytics.** Funnel, response rate, time to interview, an application heatmap, and **career lanes**: AI groups your roles by function and shows which lanes get interviews and which get rejected without one, always compared with your own average.
- **Company watchlist.** A lightweight list of companies to keep in mind, with optional AI suggestions.
- **Leaderboards.** Private groups with a blended score that favours fit over volume.
- **SQL practice.** An in-browser SQLite copy of your own jobs with guided challenges. Nothing is sent to a server.
- **Excel export and import**, to move jobs between accounts.
- **MCP server.** Claude Desktop or Claude Code can list jobs, read pipeline stats and add a job, with a two-step confirmation on writes. See [MCP tools](docs/mcp-tools.md).

## Built with

| Layer | Technology |
|---|---|
| Frontend | React, TypeScript, Vite |
| Backend | FastAPI, async SQLAlchemy, Pydantic |
| Database | SQLite |
| AI | Google Gemini and OpenAI (the user's own key), behind one client with retries and model fallback |
| Job data | Adzuna API, Playwright for generic careers pages |
| Integration | Model Context Protocol (MCP) server with OAuth 2.1 |
| Auth | JWT, Argon2 password hashes |
| Development | Built with Claude Code, from a written specification and data model |

## Design choices

- **One structured call, not an agent loop.** Job analysis first ran as a multi-step agent loop, which hit API rate limits and gave inconsistent results. I replaced it with one structured call per job, validated in code: cheaper, faster and repeatable.
- **Consent before any AI call.** Every request that would send text to an AI provider passes one gate on the server. Users agree once, in plain language, and can withdraw at any time. Without consent the rest of the app still works.
- **Minimum data to the model.** Career lanes are built from job *titles only*. No CV, notes or company names leave the server for that feature.
- **Users' own API keys.** Each user runs AI analysis on their own key, which keeps costs predictable and separate.
- **Resilient AI client.** Retries with backoff, per-model settings and a fallback chain, with outages reported as 503 with `Retry-After` instead of generic errors.
- **Numbers come from code, not from AI.** Insights and statistics are computed deterministically so they cannot be invented.
- **Data ownership.** Everything is scoped per user, tested so one account cannot read another's jobs, and deleted immediately with the account. Users can export their own data.
- **Two-step writes for AI agents.** The MCP write tool previews first and only writes on an explicit second call.

## Limitations and what I'd change next

- **Database:** SQLite keeps hosting simple but limits concurrent writes. At larger scale I'd move to Postgres.
- **Careers-page scraping:** generic HTML scraping breaks when sites change layout. I'd prioritise official job APIs and alert on parse failures.
- **Operations:** it's self-hosted without uptime monitoring or automated tests on every change. Both would come before a wider launch.

## Screenshots

![Job detail: AI match score, hard requirements, skill gaps and interview questions](docs/screenshots/Job-Detail.png)
![Pipeline pulse: live applications and which ones need a follow-up](docs/screenshots/Analytics-1.png)
![Analytics: funnel, response rate, fit score by outcome and activity heatmap](docs/screenshots/Analytics-2.png)
![Discover: jobs from Adzuna and followed companies, ranked against the profile](docs/screenshots/Discover-Adzuna-customCompany.png)
![Claude listing and adding jobs through the MCP server](docs/screenshots/MCP-Claude-in-work.png)

## Read more

- [Architecture](docs/architecture.md), with diagrams and request flows
- [MCP tools](docs/mcp-tools.md)
- [Data model](docs/data-model.md)

## About

Built by Abhishek Anand as a hands-on project in full-stack development, applied AI and agent integration, using Claude Code. https://github.com/meVky/