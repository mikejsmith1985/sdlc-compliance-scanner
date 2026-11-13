# SDLC Compliance Scanner

Scans GitHub Projects boards against a SOC 2–aligned SDLC, flags tickets that may be out of compliance, and suggests a concrete fix for each finding. It has two views: an executive overview of how teams are doing, and a ticket workbench for working through findings.

> **Status: clickable prototype, mock data only.** The tickets, teams, rules, thresholds and SOC 2 criteria tags in `prototype/` are illustrative placeholders. Nothing connects to GitHub yet.

## Run the prototype

No build, no install, no network needed.

1. Clone the repo, or download it as a ZIP.
2. Open `prototype/index.html` in any modern browser (Edge, Chrome, Firefox).

Try this demo flow:

1. On **Executive overview**, click one of the oldest open findings.
2. In **Ticket workbench**, click **Mark fixed** or **Document exception**. An exception requires a written justification.
3. Switch back to **Executive overview**. The compliance rate, trend, team table and counts update.

State lives in memory only. Reload the page to reset it.

## What it does

| View | For | Shows |
|---|---|---|
| Executive overview | Leadership | Compliance rate and 8-week trend, open findings by SDLC gate, results by team, oldest open findings |
| Ticket workbench | Delivery leads / scrum masters | Filter by team, severity and status. Per ticket: what the scan found, suggested fix, criteria tag, actions (assign, mark fixed, document exception, reopen) and an audit trail |

## Example rules (placeholders)

These rules are examples only. The real list will come from the team's SDLC. They're defined in `rules/sdlc-rules.example.yaml`.

| ID | Rule | Gate | Criteria tag* | Severity |
|---|---|---|---|---|
| R01 | Done without a merged PR | Traceability | CC8.1 | High |
| R02 | No independent code review | Peer review | CC8.1 | High |
| R03 | Merged with failing or skipped checks | Testing & CI | CC8.1 | High |
| R04 | Missing acceptance criteria | Requirements | CC8.1 | Medium |
| R05 | Risk rating not set | Risk assessment | CC3.2 | Low |
| R06 | Emergency change without post-implementation review | Emergency change | CC8.1 | High |
| R07 | Security finding past remediation SLA | Vulnerability mgmt | CC7.1 | Critical |
| R08 | Deployed without release approval | Release approval | CC8.1 | Medium |

\*The mapping to SOC 2 Trust Services Criteria (2017 TSC, 2022 points of focus) is a working assumption. Confirm it with our security and compliance owners (whoever works with the SOC 2 auditors). The AICPA has also proposed revised criteria; the exposure draft was published in October 2026, so re-check the mapping once that's finalized.

## Planned architecture (POC)

```
GitHub (Projects v2, Issues, PRs, reviews, checks, deployments, branch protection)
        │  read-only token
        ▼
  Collector ──► dated snapshot (JSON)   ← snapshots are audit evidence over time
        │
        ▼
  Rules engine  ← rules/*.yaml (SDLC gates → criteria, severity, fix template)
        │
        ▼
  Findings store (status, owner, exceptions + justification, audit trail)
        │
        ▼
  Dashboard (overview + workbench)
```

Design principles:
- **Read-only against GitHub.** The scanner never changes tickets or repos.
- **Deterministic fixes.** Suggested fixes come from templates in the rules file, so the same finding always gets the same advice. Any AI-written wording is an optional extra layer, not the source of truth.
- **Exceptions need a justification and an approver.** That trail is itself audit evidence.
- **Flags, not verdicts.** The tool surfaces likely gaps. The auditor decides what counts as an exception.

## Roadmap

- [ ] Replace the example rules with the real SDLC gates
- [ ] Collector: Projects v2 items plus linked PRs, reviews and checks (GitHub GraphQL and REST)
- [ ] Rules engine reading `rules/*.yaml`
- [ ] Persist snapshots and findings
- [ ] Wire the prototype UI to real findings
- [ ] Auth and hosting approved by our security and compliance owners

## Security and data handling

Get approval from our security and compliance owners before you run this against real repos. In particular:
- Where the tool runs, and what token scopes it uses (read-only, least privilege).
- That no secrets are committed. `.env` files are git-ignored; use `.env.example` for templates.
- That ticket content may contain sensitive data. Don't send it to outside services without approval.

## Repo layout

```
prototype/index.html            clickable mock-data prototype (single file, no dependencies)
rules/sdlc-rules.example.yaml   example rule definitions (placeholders)
docs/                           design notes
```
