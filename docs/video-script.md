# Walkthrough video: narration script

The narration for [`media/SDLC-Compliance-Scanner-demo.mp4`](../media/SDLC-Compliance-Scanner-demo.mp4), by section. Use it as talking points when presenting the demo live.

## 1. Introduction

This is a walkthrough of the SDLC Compliance Scanner prototype. Using mock data, it shows how we could keep our delivery work audit-ready all year, instead of scrambling at audit time. Think of it as how this would work in theory. At the end, I'll cover where it's headed, and what a formal proof of concept would need to decide.

## 2. The problem

Here's the problem it solves. SOC 2 auditors don't just check that we have an SDLC. For a Type Two report, they test whether we actually followed it, ticket by ticket, over a period of months. Today, gaps like a missing code review or an unapproved release usually surface late, when they're harder to fix, and the evidence is harder to find.

## 3. How it works

Here's how it would work, in four steps. First, it reads our GitHub Projects boards, along with the linked pull requests, reviews, test checks, and deployments. Second, it checks each ticket against the criteria for its current status. A ticket in refinement isn't expected to have test evidence yet, but a ticket moving to Done should. Third, anything that fails becomes a finding, with a severity and a specific suggested fix. Findings would be tracked as GitHub issues in a dedicated compliance project, so we aren't building a separate ticketing system. And fourth, every scan saves a dated snapshot, building a history of evidence over time.

## 4. Executive overview: headline numbers

This is the executive overview. Across the top: our compliance rate, meaning the share of tickets with no open findings. How many findings are open, and how many of those are critical or high. How long they've been sitting. And what's been resolved.

## 5. Trend and SDLC gates

Below that, the trend shows whether we're improving week over week, against a target. And this panel shows which SDLC gates are slipping most, so we know whether the issue is code review, testing, approvals, or something else.

## 6. By team and oldest findings

The team table shows where to focus. And the oldest findings list surfaces anything that's been sitting too long. One click takes you straight into the details.

## 7. Ticket workbench

This is the ticket workbench, where delivery leads do the actual clean-up. Each flagged ticket shows exactly what the scan found, which SDLC rule it breaks, and a concrete fix. Here, a security fix has been open past its deadline. The suggestion is to patch it and link the clean re-scan, or file a formal risk acceptance.

## 8. Actions and audit trail

From here, someone can take ownership, mark it fixed, or document an exception. In the real version, these actions would happen on the finding's GitHub issue, so its history becomes the audit trail. And marking something fixed doesn't close it on someone's word. The next scan has to confirm the evidence.

## 9. Exceptions

Exceptions are handled in the open. If something can't be fixed right now, the tool requires a written justification, and flags it for approver sign-off. That justification becomes evidence too.

## 10. Overview updates

And because it's all connected, the overview updates as findings are resolved. Leadership sees the numbers move, without anyone building a status report.

## 11. Guardrails

A few guardrails are built into the design. It doesn't change the teams' tickets or code. It only writes to its own compliance project. Suggested fixes come from fixed templates, so the advice is consistent and explainable. It flags likely gaps, but the auditor still makes the final call. And the rules, thresholds, and SOC 2 mappings shown here are placeholders, to be confirmed with our security and compliance owners.

## 12. Target state

Where this is headed is an automated identification and reconciliation process, built into our workflow. Every status on the board has its own gate: the criteria a ticket should meet before it moves on. When a ticket changes status and misses its gate, an AI agent could step in, but within firm limits. It can fix gaps it can prove from system records, like linking the merged pull request, or attaching the passing test run. And it comments on the ticket, so every correction is visible. What it never does is create evidence that a person was supposed to provide. Missed code reviews, testing, approvals, and sign-offs get flagged, and a person handles them.

## 13. Formal POC request

A formal proof of concept would need a few decisions up front. Which SDLC gates apply at each status. Where findings are tracked, and what access the scanner gets. Whether issue history satisfies our auditors, or we also keep tamper-resistant snapshots. And, before any AI touches tickets, an approved model provider for the data that may be in them. Then we phase it. Version one scans and flags only, to prove the rules are accurate. Version two adds status-change checks, and an agent that drafts fixes for a person to approve. Version three auto-applies the safe fixes, and that's where an agent framework like LangGraph earns its place. To start, we need our real SDLC gates, sign-off from our security and compliance owners, and one or two pilot teams. The goal: less admin for engineering managers, and no surprises at audit time.
