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
| **Current phase** | Phase II — Reproduction & Solution Planning |
| **Progress summary** | Cloned the fork and installed shields locally; traced the bug to the Azure DevOps **build** badge extending the no-auth scraper base class instead of the authenticated API base class its siblings use, and pinned the root cause to specific files/lines; wrote a UMPIRE migration plan. |
| **Deliverable links** | [Issue #10162](https://github.com/badges/shields/issues/10162) · [Fork](https://github.com/azizu06/shields) · [Branch `fix-issue-10162`](https://github.com/azizu06/shields/tree/fix-issue-10162) · [Reproduction commit](https://github.com/azizu06/shields/commit/fce9f6a9868551e3c545860c2f6f28bc195c257e) · [Plan commit](https://github.com/azizu06/shields/commit/1ce3d503e692e158180769b974fe4169b4baf5d9) |
| **Blockers / questions** | None blocking. One open design question: replicate the `stage`/`job` filters via Azure's Timeline API, or scope them out of this PR — will confirm with the maintainer. |

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

- [x] Entered in cohort issue ledger

---

## Phase II — Reproduction & Solution Planning

> **Completion signal:** Forked repo with 2+ starter commits — a reproduction comment and a markdown project plan

### Repository Fork

- **Fork URL:** https://github.com/azizu06/shields
- **Local setup completed:** **Yes** — cloned the fork, `npm ci` completed with no errors, dev server runs.

### Reproduction

**Environment:**
- **OS:** macOS (Apple Silicon), Darwin 25.5
- **Language / runtime version:** Node.js **v24.15.0** (via nvm), npm 11.x
- **Repo:** `badges/shields` (default branch `master`); fork `azizu06/shields`

**Setup notes / challenges (real issues I hit and fixed):**
1. shields' default branch is **`master`**, not `main`.
2. `npm ci` on **Node 22.19 fails** with `EBADENGINE` — `.npmrc` sets `engine-strict=true` and the dev dependency `lint-staged` requires Node `>=22.22.1`. Fixed by switching to **Node 24** (`nvm use 24`).
3. Native modules (e.g. `re2`) are compiled per Node version, so the badge server must **run on the same Node version used to install**, or it crashes at startup with a `NODE_MODULE_VERSION` mismatch (`ERR_DLOPEN_FAILED`). Fixed by running the server on Node 24 too.

**Why this reproduction is code/behavior based:** The reported symptom is "the build badge fails on a *private* Azure DevOps project even with a PAT configured." Reproducing that *literally* requires a private Azure org with a completed pipeline and a PAT — and Microsoft gates free CI parallelism behind a manual grant that can take several business days. The faithful, self-contained reproduction is therefore to demonstrate the *mechanism*: the build badge never attaches the configured PAT, while its sibling badges do.

**Steps to reproduce:**

```
Method A — code inspection (proves the missing-auth path; fully reproducible now):
1. Open services/azure-devops/azure-devops-build.service.js. Line 38:
   `class AzureDevOpsBuild extends BaseSvgScrapingService` — the SVG-scraper base, which
   carries no auth config.
2. In handle() (lines 110–132) it fetches
   https://dev.azure.com/{org}/{projectId}/_apis/build/status/{definitionId}
   via the helper imported on line 9.
3. Open services/azure-devops/azure-devops-helpers.js, lines 16–18: the helper calls
   serviceInstance._requestSvg(...) with NO authHelper — so no Authorization header is sent.
4. Contrast services/azure-devops/azure-devops-coverage.service.js line 44
   (`extends AzureDevOpsBase`) and azure-devops-base.js lines 16 + 23–24, which declare
   `static auth = { passKey: 'azure_devops_token' }` and fetch with
   this.authHelper.withBasicAuth(...) — the PAT IS attached.

Method B — local runtime confirmation (to be captured as the reproduction commit):
5. Start shields locally: `npm start` (badge server + docs frontend; the badge URL/port is
   printed in the terminal).
6. Render the build badge for the public example project from the badge's own docs:
   /azure-devops/build/totodem/8cf3ec0e-d0c2-4fcd-8206-ad204f254a96/2.svg
   -> renders `build | passing` and works ANONYMOUSLY (the example project is public).
7. Configure a PAT via the `azure_devops_token` secret and re-request the same badge with
   request logging (`npm run test:services:trace`) — the outbound request to
   /_apis/build/status/... carries NO Authorization header, confirming the PAT is ignored
   on the build path. (A nock-based unit test asserting "no auth header sent" will be the
   committed reproduction artifact.)
```

**Observed behavior:** The Azure DevOps build badge fetches its data through `BaseSvgScrapingService._requestSvg` with no authentication. Even when `azure_devops_token` is configured, no `Authorization` header is sent, so any project that blocks anonymous access returns an error and the badge fails.

**Expected behavior:** Like the coverage / tests / release badges, the build badge should send the configured PAT (HTTP basic auth) so it can read build status for private projects.

**Reproduction commit:** [`fce9f6a`](https://github.com/azizu06/shields/commit/fce9f6a9868551e3c545860c2f6f28bc195c257e) — adds [`codepath/reproduction.md`](https://github.com/azizu06/shields/blob/fix-issue-10162/codepath/reproduction.md)

### Root Cause Analysis

The bug is **structural**, not a missing line inside one function. In shields, auth is attached based on *which base class a service extends and which `_request*` method it calls*:

- **The build badge** (`services/azure-devops/azure-devops-build.service.js:38`) extends **`BaseSvgScrapingService`** and reads data by scraping Azure's native status *image* (`/_apis/build/status/{definitionId}`) through `azure-devops-helpers.js:18` → `_requestSvg(...)`. That path declares no `static auth` and never calls `authHelper`, so the PAT is never sent.
- **Every other Azure DevOps badge** (coverage `:44`, tests, release) extends **`AzureDevOpsBase`** (`azure-devops-base.js:15`), which declares `static auth = { passKey: 'azure_devops_token', authorizedOrigins: ['https://dev.azure.com'] }` (`:16`) and fetches via `this._requestJson(this.authHelper.withBasicAuth(...))` (`:23–24`). Those badges send the PAT and work on private projects.

The scraped image endpoint is an anonymous-only surface, so the badge can't simply have a header bolted on — it has to move onto the **authenticated JSON API**, which `AzureDevOpsBase` already partly implements: `getLatestCompletedBuildId()` (`azure-devops-base.js:33`) already queries the auth'd `/_apis/build/builds` endpoint.

- **Primary file to change:** `services/azure-devops/azure-devops-build.service.js`
- **Supporting:** `services/azure-devops/azure-devops-base.js` (reuse/extend); the scraper helper in `azure-devops-helpers.js` may become unused for builds.

### Solution Approach

Framed with the **UMPIRE** structure.

**Understand:** The Azure DevOps build badge doesn't send the user's PAT, so it breaks on private projects. It should authenticate like the other Azure DevOps badges.

**Match:** `AzureDevOpsCoverage` (`azure-devops-coverage.service.js`) is the working template — it `extends AzureDevOpsBase`, calls `getLatestCompletedBuildId(...)` (`:112`), then fetches an authenticated JSON endpoint (`:127`) and renders. The build badge should follow the same shape.

**Plan:**
1. Change `AzureDevOpsBuild` to `extends AzureDevOpsBase` so it inherits `static auth` + the authenticated `fetch`.
2. Replace the SVG scrape with a JSON call to `GET /_apis/build/builds?definitions={definitionId}&$top=1&branchName=refs/heads/{branch}` (authenticated via the base `fetch`).
3. Read `value[0].status` + `value[0].result`, map them to the existing build-status badge output (reuse `renderBuildStatusBadge` / the build-status mapping).
4. **Decide `stage`/`job`** (currently lines 12–13 and 119–120): the builds-list API returns whole-build status only. **Tradeoff —** (a) replicate per-stage/job via Azure's **Timeline API** (`/_apis/build/builds/{buildId}/timeline`, which returns records typed `Stage`/`Job` with their own `result`), or (b) scope them out of this PR and raise it with the maintainer. Current lean: **(a)** to avoid regressing existing behavior, with (b) as a fallback if the maintainer prefers a smaller PR.
5. Add `azure-devops-build.spec.js` — the build badge currently has only a `.tester.js`, no unit spec.

**Implement:** _(Phase III)_ — on branch `fix-issue-10162` in `azizu06/shields`.

**Review:** Self-review against shields' `CONTRIBUTING.md`: PRs are squash-merged (no need to pre-squash my own commits); **service changes must include tests**; the **PR title must tag the affected service in square brackets** so CI runs those service tests — e.g. `[AzureDevops] Use PAT auth for the build badge on private projects`. Commit messages/authorship are not amendable after merge.

**Evaluate:** shields tests two ways and I'll use both — service tests (`azure-devops-build.tester.js`, run with `npm run test:services -- --only=azure-devops`, must keep ≥1 live "picture check") and unit tests (`*.spec.js` using `nock` to fake Azure). Specifically I'll: (1) add a `nock` unit test asserting `Authorization: Basic …` **is** sent when a token is configured (the inverse of the reproduction); (2) add unit tests mapping Azure `result` values (`succeeded` / `failed` / `canceled` / `partiallySucceeded`) to badge output; (3) keep a live picture-check test green; (4) run `npm test` to confirm no regressions. **No real Azure account needed** — `nock` supplies the fake responses.

**Implementation plan commit:** [`1ce3d50`](https://github.com/azizu06/shields/commit/1ce3d503e692e158180769b974fe4169b4baf5d9) — adds [`codepath/plan.md`](https://github.com/azizu06/shields/blob/fix-issue-10162/codepath/plan.md)

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
