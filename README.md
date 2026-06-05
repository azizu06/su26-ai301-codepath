# AI 301 Contribution README

**Name:** Abduaziz Umarov  
**GitHub:** [@azizu06](https://github.com/azizu06)  
**Course:** CodePath AI 301 — AI Open Source Capstone (Summer 2026)  
**Cohort:** SU26

---

## Weekly Status

> Mentors read this section first. Keep it current — updated every Sunday by 11:59pm PST.

| Field | This Week |
|---|---|
| **Current phase** | Phase I — Issue Selection |
| **Progress summary** | Selected shields/shields issue #10162 (Azure DevOps build badge missing PAT auth), forked the repository, and commented on the issue to signal intent. |
| **Deliverable links** | [Issue #10162](https://github.com/badges/shields/issues/10162) · [Fork](https://github.com/azizu06/shields) |
| **Blockers / questions** | None |

---

## Phase I — Issue Selection

> **Completion signal:** Issue link + problem summary + cohort ledger entry

### Selected Issue

- **Repository:** [badges/shields](https://github.com/badges/shields)
- **Issue URL:** https://github.com/badges/shields/issues/10162
- **Issue title:** Azure DevOps - Build Badge - PAT Token not used on private projects
- **Labels / tags:** `good first issue`, `service-badge`

### Problem Summary

Shields.io generates live status badges for GitHub READMEs by fetching data from third-party APIs. It already supports Personal Access Token (PAT) authentication for several Azure DevOps badge types — coverage, test results, work items — but the build badge handler was written without PAT support. This means that when an organization's Azure DevOps instance is configured to block anonymous API requests (a standard enterprise security policy), the build badge silently fails while every other Azure DevOps badge on that same project works fine. The fix is to add PAT authentication to the build badge handler, following the pattern already established by the other Azure DevOps badge implementations in the same codebase.

### Why I Chose This Issue

The fix path is clearly defined — the codebase already has working PAT authentication in neighboring Azure DevOps service files, so the work is understanding that existing pattern and applying it consistently to the build badge. The project is JavaScript/Node.js, which aligns with my stack. Shields.io is one of the most widely-used developer infrastructure tools in open source, which means a merged contribution has real visibility and portfolio value. The issue is labeled `good first issue`, has no assignee, and has no open pull requests — it is available, scoped appropriately for a 3–4 week cycle, and the project has strong contributor documentation and an active community.

### Cohort Issue Ledger Entry

- [ ] Entered in cohort issue ledger

---

## Phase II — Reproduction & Solution Planning

> **Completion signal:** Forked repo with 2+ starter commits — a reproduction comment and a markdown project plan

### Repository Fork

- **Fork URL:** https://github.com/azizu06/shields
- **Local setup completed:** No — Phase II

### Reproduction

**Environment:**
- OS:
- Language / runtime version:
- Steps to reproduce:

```
1.
2.
3.
```

**Observed behavior:**

**Expected behavior:**

**Reproduction commit:** _(link to commit where you documented the reproduction)_

### Root Cause Analysis

_What did you find when tracing the code? Where does the bug originate or where does the missing feature need to be added? Be specific — file names and line numbers where applicable._

### Solution Approach

_Describe your implementation plan. What will you change, add, or remove? What are the tradeoffs of this approach vs. alternatives you considered?_

**Implementation plan commit:** _(link to commit with your markdown plan)_

---

## Phase III — Solution Building

> **Completion signal:** WIP branch with active daily commits

### WIP Branch

- **Branch URL:**
- **First commit date:**
- **Most recent commit date:**

### Implementation Notes

_Running notes as you build. Update this as you make progress — what worked, what you had to change, and why._

| Date | Note |
|---|---|
| | |

### Testing Strategy

_What tests are you writing? Unit tests, integration tests, or both? What edge cases are you covering?_

**Test file(s):**

### Mentor Feedback Requests

_Document any feedback you requested and received during this phase._

| Date | Question / request | Response |
|---|---|---|
| | | |

---

## Phase IV — Pull Request & Submission

> **Completion signal:** Submitted PR + evidence of maintainer communication

### Pull Request

- **PR URL:**
- **PR title:**
- **Submitted date:**
- **Status:** Open / Changes requested / Merged

### PR Description Summary

_Summarize what your PR does, what issue it closes, and how you tested it. This mirrors what you wrote in the actual PR description._

### Pre-submission Checklist

- [ ] Code addresses the selected issue
- [ ] Tests written and passing
- [ ] Documentation follows the project's contribution guidelines
- [ ] PR description explains the change clearly
- [ ] Self-reviewed against the project's CONTRIBUTING.md

### Maintainer Feedback Log

_Log every round of review feedback and your response. This is evidence of professional iteration._

| Date | Reviewer | Feedback | My response / commit |
|---|---|---|---|
| | | | |

---

## Contribution Cycle Log

_If you complete a full cycle and start a second one, add a new section above and archive the previous cycle here with a summary._

| Cycle | Issue | PR | Outcome |
|---|---|---|---|
| 1 | | | |
