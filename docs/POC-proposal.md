# Proposal: SDLC Compliance Scanner — formal proof of concept

**Status:** Draft for discussion · **Author:** Mike Smith · **Date:** November 2026

## 1. Summary

The prototype in this repo shows, with mock data, how we could keep delivery work audit-ready all year instead of reconstructing evidence at audit time. It explains how the tool would work in theory. It doesn't connect to GitHub yet.

This document asks for a formal proof of concept (POC). It covers:
- the target state we're aiming for
- the architecture decisions the POC has to make
- the concerns we already know about
- a phased plan
- what we need from leadership to start

## 2. The problem

A SOC 2 Type II report tests whether our controls actually operated over a review period, not just whether a documented SDLC exists. For delivery teams, that means auditors sample tickets and check the evidence: a merged PR, an independent review, passing tests, an approval before release.

Today, gaps such as a missing review, a skipped check or an unapproved release tend to surface late. By then they're harder to fix, the evidence is harder to find, and engineering managers spend time on manual evidence chasing.

## 3. What the demo shows (in theory)

| View | Audience | What it shows |
|---|---|---|
| Executive overview | Leadership | Compliance rate and trend, open findings by SDLC gate, results by team, oldest open findings |
| Ticket workbench | Delivery leads | What the scan found, which rule it breaks, a suggested fix, actions (assign, fix, document an exception), and an audit trail |

Everything in the demo is mock data. That includes the rules, thresholds and SOC 2 criteria tags, which are placeholders.

## 4. Target state

### 4.1 Automated identification and reconciliation, built into the workflow

- **Gates per status.** Each board status (for example Refinement, Ready, In Progress, In Review, Done) maps to the SDLC and SOC 2 criteria a ticket must meet before it moves on. A ticket in Refinement isn't flagged for missing test evidence; a ticket moving to Done is.
- **Scan on status change.** Each status change triggers a check against the new status's gate, with a scheduled sweep as a backstop. *To verify:* GitHub Projects may not be able to block a status change the way a Jira workflow can. If not, the control is detective (it flags right after the move) rather than preventive.
- **Reconciliation.** Each scan opens, updates, closes or reopens findings so they match reality. A finding closes only when a later scan confirms the evidence.

### 4.2 Where AI helps, and where people decide

| An AI agent **may** | An AI agent **must never** |
|---|---|
| Fix gaps it can prove from system records: link the merged PR that clearly belongs to the ticket, attach the passing check run, fill a field from data that already exists | Create or imply evidence of something a person was supposed to do: code reviews, testing, approvals, risk sign-offs |
| Comment on the ticket explaining what it changed and why | Edit history or backdate anything |
| Draft a suggested fix for a person to approve | Close a finding without a confirming scan |

- **Coding or testing misses are never auto-corrected.** They're flagged, and a person handles them.
- **Every agent edit is visible as an after-the-fact correction.** It's posted as a bot comment and logged, so auditors see remediation, not evidence that appears to have been there all along.

### 4.3 Where findings live

Findings are tracked as **GitHub issues in a dedicated compliance repo and Project**, not in a new ticketing system:
- Each finding is one issue, linked back to the source ticket.
- Labels and Project fields hold the rule, severity, team and status.
- Actions such as assign, comment, close and document an exception happen in GitHub, so the issue timeline is the audit trail.
- The dashboard is a reporting view over that Project.

## 5. Architecture decisions for the POC

| # | Decision | Options | Starting recommendation |
|---|---|---|---|
| D1 | SDLC gates per status | Which criteria apply at which status; which SOC 2 criteria each one supports | Derive them from our SDLC with the SDLC owner, then confirm the SOC 2 mapping with our security and compliance owners |
| D2 | Where findings live | (a) issues in a dedicated compliance repo and Project; (b) comments or labels on the source tickets; (c) a separate database | (a). Optionally add a light-touch comment on source tickets later |
| D3 | Access model | Read-only on team repos plus write on the compliance repo only; or broader write | Least privilege: read on team repos, write only to the compliance repo for v1 |
| D4 | Trigger model | Scheduled scan; webhook on status change; both | Scheduled scan in v1; add status-change triggers in v2 |
| D5 | Evidence integrity | Issue history alone; plus dated, tamper-resistant scan snapshots | Keep snapshots too, since issue history can be edited or deleted by admins. Confirm with our security and compliance owners what auditors will accept |
| D6 | Where it runs | GitHub Actions in our org; an internal service; other | Decide with security. Must meet internal hosting and secrets standards |
| D7 | Rules engine | Hard-coded checks; declarative rules file | Declarative rules (see `rules/sdlc-rules.example.yaml`), versioned and reviewed like code |
| D8 | AI model and data handling (v2+) | Which model provider; what ticket data may be sent | Only an approved provider for data that may appear in tickets, which may include sensitive or patient data (in practice, typically with a business associate agreement) |
| D9 | Orchestration (v3) | Simple scripts; an agent framework such as LangGraph | Not needed for v1. Reconsider when multi-step flows with human approval steps appear |

## 6. Concerns and risks

- **Evidence integrity.** Automated corrections must never look like backdated or invented evidence (see 4.2).
- **Sensitive data.** Tickets may contain sensitive or patient data. Any AI processing needs an approved provider and minimal data sharing.
- **False positives.** If the rules are noisy, teams will ignore findings. v1 exists to prove rule accuracy before anything acts automatically.
- **Control design.** Flagging after a status change is a detective control, not a preventive one. Confirm that's acceptable, or decide where a hard block is needed.
- **Ownership.** Someone has to own the rules, triage findings and approve exceptions. Name them before rollout.
- **Access scope.** Even read access to many repos needs approval. Write access is limited to the compliance repo.
- **Changing standards.** The AICPA has proposed revised SOC 2 Trust Services Criteria (exposure draft, October 2026). The rule-to-criteria mapping will need review once that's finalized.

## 7. Phased plan

| Phase | Scope | Exit criteria |
|---|---|---|
| **Demo** (done) | Clickable prototype with mock data; this proposal | Leadership agrees to a POC |
| **POC v1** | Scheduled scan of 1–2 pilot teams' boards; fixed rules; findings opened and reconciled as GitHub issues; dashboard reads from the compliance Project; **flag only** | Rule accuracy agreed with pilot teams; false-positive rate acceptable; evidence approach accepted by our security and compliance owners |
| **v2** | Status-change triggers; an agent that **drafts** fixes for provable gaps, with a person approving | Drafted fixes reliably correct; no human-only gate ever touched |
| **v3** | Auto-apply safe, provable fixes; human approval steps for everything else; consider LangGraph or a similar harness | Measurable reduction in manual evidence work; clean audit sampling |

**Suggested POC success measures** (targets to be agreed):
- the share of pilot-team tickets meeting their gates
- the median time to resolve a finding
- the false-positive rate
- engineering manager hours spent on evidence collection, before and after

## 8. What we need to start

1. **Our SDLC's real gates**, mapped to board statuses, from the SDLC owner.
2. **Sign-off from our security and compliance owners** on the access model, where it runs, and the evidence approach (D3, D5, D6).
3. **One or two pilot teams** willing to run it on their boards.

## 9. Open questions

- Who works with our SOC 2 auditors, and what evidence format do they prefer?
- Are our boards' status columns consistent enough across teams to map gates once, or will each team need its own mapping?
- Do any teams track work outside GitHub Projects (for example in Jira) that would also need coverage?
