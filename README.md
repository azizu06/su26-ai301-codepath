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
| **Current phase** | Cycle 2 · Phase II — Reproduction and Solution Planning (technical artifacts complete, Course Portal check-in pending) |
| **Progress summary** | Reproduced issue #11286 against the live `badges/shields` PR badge. The normal request and `?excludeDrafts` both returned 26 open PRs while GitHub search showed 19 non-drafts and 7 drafts, which confirmed that Shields ignores the proposed parameter today. I traced that behavior through `GithubIssues.route`, `handle()`, and `fetch()`, then wrote a UMPIRE plan around the maintainer-requested `GitHubIssues` / `GitHubPullRequests` split. |
| **Deliverable links** | [Phase II branch](https://github.com/azizu06/shields/tree/issue-11286-docs) · [Reproduction commit `46dc6bc177`](https://github.com/azizu06/shields/commit/46dc6bc177) · [Reproduction document](https://github.com/azizu06/shields/blob/issue-11286-docs/codepath/reproduction.md) · [Plan commit `d3a6b7ea0b`](https://github.com/azizu06/shields/commit/d3a6b7ea0b) · [UMPIRE plan](https://github.com/azizu06/shields/blob/issue-11286-docs/codepath/plan.md) · [Issue #11286](https://github.com/badges/shields/issues/11286) · [Maintainer roadmap](https://github.com/badges/shields/pull/11401#issuecomment-3692559871) |
| **Blockers / questions** | No technical blockers. I still need to submit the Course Portal check-in with Phase II Complete marked. Before Phase III I also want to confirm whether maintainers prefer mutually exclusive validation when both `excludeDrafts` and `onlyDrafts` are supplied. |

---

## Cycle 2 — Issue #11286 (current)

### Phase I — Issue Selection

> **Completion signal:** Issue link + problem summary + cohort ledger entry

#### Selected Issue

- **Repository:** [badges/shields](https://github.com/badges/shields)
- **Issue URL:** https://github.com/badges/shields/issues/11286
- **Issue title:** Option to exclude drafts from PR counts
- **Labels / tags:** `service-badge`

#### Problem Summary

shields' GitHub issues-or-pull-requests badge (the badge behind https://shields.io/badges/git-hub-issues-or-pull-requests) counts open pull requests but includes drafts in the total. There's no way to leave drafts out, so the count can overstate how many pull requests are actually ready for review.

The issue as written asks for "an option to exclude drafts," but the maintainer reshaped the real scope during review of the prior attempt (see the Maintainer Guidance section below). The wanted change is to **split the combined `GitHubIssues` badge into two badges**: `GitHubIssues` keeps the issue variants (`issues`, `issues-raw`, `issues-closed`, `issues-closed-raw`) on the existing `repository` query, and a new `GitHubPullRequests` badge holds the PR variants (`issues-pr`, `issues-pr-raw`, `issues-pr-closed`, `issues-pr-closed-raw`) plus the new `excludeDrafts` / `onlyDrafts` options, backed by the GitHub `search` query. Reasoning: only PRs can be drafts, so a draft option belongs only on a PR badge; and only the `search` query supports a `draft:` filter, so counting non-draft PRs is impossible on the `repository` query the badge uses today.

#### Why I Chose This Issue

It's the highest-ROI follow-up to my first contribution. It enhances an existing, heavily used GitHub badge rather than adding a brand-new one, so the infrastructure and API path are already proven. It's also adjacent to the work I just shipped on the Azure DevOps badge (changing how an existing badge counts and filters what it reports), so I can reuse that experience directly. The issue is recent, unassigned, and had no open PR when I claimed it.

**Scope note (updated this week):** at selection time I read this as a small optional-param add. The maintainer's review reshaped it into a badge split (see below), so it's a larger, more architectural task than Cycle 1. I'm keeping it: the point of this cycle is to learn, and a badge split with an API-query migration teaches more than another param add — while still being well-scoped because the maintainer already laid out the target design.

#### Maintainer Guidance & Reshaped Scope

The maintainer's [reply on the issue](https://github.com/badges/shields/issues/11286#issuecomment-4828059214) pointed me to a prior, now-closed attempt, [PR #11401](https://github.com/badges/shields/pull/11401) by feloy, and told me to use its review comments as the starting point. Reading that PR and its review is where the real requirements live:

- feloy's original PR added `excludeDrafts` and `onlyDrafts` params onto the existing combined badge. The maintainer didn't reject the idea, he rejected the shape, then the PR went stale for ~6 months and was auto-closed on 2026-06-27.
- The maintainer's [roadmap comment](https://github.com/badges/shields/pull/11401#issuecomment-3692559871) is the actual spec: "draft issues" aren't a real concept, so don't add draft terminology to a badge that also counts issues. Instead **split** the badge — `GitHubIssues` stays as-is on the `repository` query; a new `GitHubPullRequests` badge takes the PR variants and the new draft options and uses the `search` query for all of them.
- Why `search` and not `repository`: the `repository` query returns a fixed pre-aggregated count with no way to condition on draft state. The `search` query accepts qualifiers (`is:pr is:open draft:false` / `draft:true`), so it's the only one that can count non-draft PRs. Putting *all* PR cases on `search` (not just the filtered ones) keeps a single source of truth and avoids two code paths drifting.
- **Attribution:** the maintainer asked whoever picks this up to fetch feloy's commits (`git fetch origin pull/11401/head:pr-11401`) and include a co-author trailer crediting feloy. I'll do that on the eventual PR.

#### Cohort Issue Ledger Entry

- [ ] Entered in cohort issue ledger

### Phase II — Reproduction & Solution Planning

> **Completion signal:** Forked repo with 2+ starter commits — a reproduction comment and a markdown project plan

#### Repository Fork

- **Fork URL:** https://github.com/azizu06/shields (existing fork, reused from Cycle 1)
- **Working branch:** https://github.com/azizu06/shields/tree/issue-11286-docs
- **Local setup completed:** **Yes.** I fetched the latest `upstream/master`, created a separate worktree for issue #11286, switched to Node.js v24.15.0 through nvm, and ran `npm ci` successfully.
- **Setup approach:** I reused the fork from Cycle 1 but did not reuse its working branch. The existing checkout was still on the merged `fix-issue-10162` branch and had unrelated untracked files, so I created `issue-11286-docs` from current upstream commit `aa75efb72e`. This keeps the Phase II course files out of both the old branch and the eventual code PR.
- **Setup challenges:** `npm ci` completed, but printed upstream dependency deprecation warnings and reported existing audit findings. I left those untouched because dependency upgrades are outside issue #11286. The separate worktree also avoids accidentally mixing the generated files from my first contribution cycle into this one.

#### Reproduction

I reproduced the issue using the live `badges/shields` badge and GitHub search on July 12, 2026.

1. Requested `/github/issues-pr/badges/shields.json` and got `26 open`.
2. Requested the same badge with `?excludeDrafts` and still got `26 open`.
3. Queried GitHub search for the same repository and state. It reported 7 open drafts and 19 open non-drafts.

**Observed behavior:** The normal request and the request with `?excludeDrafts` returned the same count even though the repository had open draft PRs.

**Expected behavior:** The unfiltered badge should report all 26 open PRs. The request with `?excludeDrafts` should report only the 19 open non-draft PRs.

The exact live counts can change, so the repeatable check is that the two badge responses stay equal while GitHub search confirms at least one open draft.

**Full reproduction:** [`codepath/reproduction.md`](https://github.com/azizu06/shields/blob/issue-11286-docs/codepath/reproduction.md)

**Reproduction commit:** [`46dc6bc177`](https://github.com/azizu06/shields/commit/46dc6bc177)

#### Root Cause Analysis

The current combined service never processes a draft filter. `GithubIssues.route` in `services/github/github-issues.service.js` declares the path variants but has no `queryParamSchema`. `handle()` receives only `variant`, `user`, `repo`, and `label`, then passes the derived `isPR` and `isClosed` flags to `fetch()`. There is no `excludeDrafts` or `onlyDrafts` value anywhere in that path.

For PR variants, `fetch()` calls GraphQL's `repository.pullRequests(states:, labels:)` field and returns `totalCount`. An open PR can also be a draft, and that connection has no draft filter, so every matching open draft stays in the total. Shields' base service passes an empty service-query-parameter object when a route has no query schema, which is why adding `?excludeDrafts` is ignored instead of changing the GitHub request.

The missing feature therefore cannot be fixed only by adding another argument to the existing `pullRequests` call. GitHub's search query supports `draft:false` and `draft:true`, so the PR variants need a search-based data path.

**Files and functions involved:**

- `services/github/github-issues.service.js`: `route`, `fetch()`, and `handle()`
- `core/base-service/base.js`: service query-parameter validation and transformation
- `services/github/github-issues-search.service.js`: analogous GraphQL search implementation
- `services/github/github-issues.tester.js`: existing issue and PR behavior tests

#### Solution Approach

I structured the full plan with UMPIRE.

- **Understand:** The badge needs to preserve every existing issue and PR URL while allowing PR counts to include only drafts or exclude drafts.
- **Match:** The maintainer's review on PR #11401 defines the target split. `github-issues-search.service.js` already demonstrates GraphQL `search(query:, type: ISSUE) { issueCount }`, and the prior PR contains useful investigation and tests that need to be reshaped rather than copied unchanged.
- **Plan:** Keep issue variants in `GithubIssues` on `repository.issues`. Move the four PR variants into a new `GithubPullRequests` service, add presence-only `excludeDrafts` and `onlyDrafts` query parameters, and use one search path for filtered and unfiltered PR counts. Preserve rendering, labels, public URLs, authentication, and repo-not-found behavior.
- **Implement:** Begin Phase III on a clean `fix-issue-11286` branch. Keep these CodePath documents off the eventual upstream PR and preserve the original contributor attribution requested by the maintainer.
- **Review:** Check route compatibility, parameter conventions, scope, attribution, the Shields contribution guide, and the PR template before submission.
- **Evaluate:** Add deterministic tests for unfiltered, `draft:false`, `draft:true`, labels, open/closed, raw/non-raw, zero matches, repo not found, and conflicting flags. Run focused tests first, then the required broader checks.

The plan also uses `git log` and `git blame`, identifies GitHub issue search as the analogous implementation, and calls out label escaping and conflicting draft flags as proactive edge cases.

**Full implementation plan:** [`codepath/plan.md`](https://github.com/azizu06/shields/blob/issue-11286-docs/codepath/plan.md)

**Implementation plan commit:** [`d3a6b7ea0b`](https://github.com/azizu06/shields/commit/d3a6b7ea0b)

#### Phase II Check-in

- [ ] Submitted the Course Portal check-in with **Phase II Complete** marked

### Phase III — Solution Building

> **Completion signal:** WIP branch with active daily commits

#### WIP Branch

- **Branch URL:**
- **First commit date:**
- **Most recent commit date:**

#### Implementation Notes

_To be completed in Phase III._

| Date | Note |
|---|---|
| | |

#### Testing Strategy

_To be completed in Phase III._

#### Mentor Feedback Requests

| Date | Question / request | Response |
|---|---|---|
| | | |

### Phase IV — Pull Request & Submission

> **Completion signal:** Submitted PR + evidence of maintainer communication

#### Pull Request

- **PR URL:**
- **PR title:**
- **Submitted date:**
- **Status:** Open / Changes requested / Merged

#### PR Description Summary

_To be completed in Phase IV._

#### Pre-submission Checklist

- [ ] Code addresses the selected issue
- [ ] Tests written and passing
- [ ] Documentation follows the project's contribution guidelines
- [ ] PR description explains the change clearly
- [ ] Self-reviewed against the project's CONTRIBUTING.md

#### Maintainer Feedback Log

_Log every round of review feedback and your response. This is evidence of professional iteration._

| Date | Reviewer | Feedback | My response / commit |
|---|---|---|---|
| | | | |

---

## Cycle 1 — Issue #10162 (Merged, archived)

### Phase I — Issue Selection

> **Completion signal:** Issue link + problem summary + cohort ledger entry

#### Selected Issue

- **Repository:** [badges/shields](https://github.com/badges/shields)
- **Issue URL:** https://github.com/badges/shields/issues/10162
- **Issue title:** Azure DevOps - Build Badge - PAT Token not used on private projects
- **Labels / tags:** `good first issue`, `service-badge`

#### Problem Summary

Shields.io generates live status badges for GitHub READMEs by fetching data from third-party APIs. It already supports Personal Access Token (PAT) authentication for several Azure DevOps badge types — coverage, test results, work items — but the build badge handler was written without PAT support. So when an organization's Azure DevOps instance blocks anonymous API requests, which is a standard enterprise security policy, the build badge silently fails while every other Azure DevOps badge on the same project keeps working. The fix is to add PAT authentication to the build badge handler, following the pattern already established by the other Azure DevOps badge implementations in the same codebase.

#### Why I Chose This Issue

The fix path is clearly defined — the codebase already has working PAT authentication in neighboring Azure DevOps service files, so the work is understanding that existing pattern and applying it consistently to the build badge. The project is JavaScript/Node.js, which aligns with my stack. Shields.io is one of the most widely used developer infrastructure tools in open source, so a merged contribution carries real visibility and portfolio value. The issue is labeled `good first issue`, has no assignee, and has no open pull requests — it is available, scoped appropriately for a 3–4 week cycle, and the project has strong contributor documentation and an active community.

#### Cohort Issue Ledger Entry

- [x] Entered in cohort issue ledger

### Phase II — Reproduction & Solution Planning

> **Completion signal:** Forked repo with 2+ starter commits — a reproduction comment and a markdown project plan

#### Repository Fork

- **Fork URL:** https://github.com/azizu06/shields
- **Local setup completed:** **Yes** — cloned the fork, `npm ci` completed with no errors, dev server runs.

#### Reproduction

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

**Reproduction commit:** [`3e940ee`](https://github.com/azizu06/shields/commit/3e940eefab) — adds [`codepath/reproduction.md`](https://github.com/azizu06/shields/blob/issue-10162-docs/codepath/reproduction.md)

#### Root Cause Analysis

The bug is **structural**, not a missing line inside one function. In shields, auth is attached based on *which base class a service extends and which `_request*` method it calls*:

- **The build badge** (`services/azure-devops/azure-devops-build.service.js:38`) extends **`BaseSvgScrapingService`** and reads data by scraping Azure's native status *image* (`/_apis/build/status/{definitionId}`) through `azure-devops-helpers.js:18` → `_requestSvg(...)`. That path declares no `static auth` and never calls `authHelper`, so the PAT is never sent.
- **Every other Azure DevOps badge** (coverage `:44`, tests, release) extends **`AzureDevOpsBase`** (`azure-devops-base.js:15`), which declares `static auth = { passKey: 'azure_devops_token', authorizedOrigins: ['https://dev.azure.com'] }` (`:16`) and fetches via `this._requestJson(this.authHelper.withBasicAuth(...))` (`:23–24`). Those badges send the PAT and work on private projects.

The scraped image endpoint is an anonymous-only surface, so the badge can't simply have a header bolted on — it has to move onto the **authenticated JSON API**, which `AzureDevOpsBase` already partly implements: `getLatestCompletedBuildId()` (`azure-devops-base.js:33`) already queries the auth'd `/_apis/build/builds` endpoint.

- **Primary file to change:** `services/azure-devops/azure-devops-build.service.js`
- **Supporting:** `services/azure-devops/azure-devops-base.js` (reuse/extend); the scraper helper in `azure-devops-helpers.js` may become unused for builds.

#### Solution Approach

Framed with the **UMPIRE** structure.

**Understand:** The Azure DevOps build badge doesn't send the user's PAT, so it breaks on private projects. It should authenticate like the other Azure DevOps badges.

**Match:** `AzureDevOpsCoverage` (`azure-devops-coverage.service.js`) is the working template — it `extends AzureDevOpsBase`, calls `getLatestCompletedBuildId(...)` (`:112`), then fetches an authenticated JSON endpoint (`:127`) and renders. The build badge should follow the same shape.

**Plan:**
1. Change `AzureDevOpsBuild` to `extends AzureDevOpsBase` so it inherits `static auth` + the authenticated `fetch`.
2. Replace the SVG scrape with a JSON call to `GET /_apis/build/builds?definitions={definitionId}&$top=1&branchName=refs/heads/{branch}` (authenticated via the base `fetch`).
3. Read `value[0].status` + `value[0].result`, map them to the existing build-status badge output (reuse `renderBuildStatusBadge` / the build-status mapping).
4. **Decide `stage`/`job`** (currently lines 12–13 and 119–120): the builds-list API returns whole-build status only. **Tradeoff —** (a) replicate per-stage/job via Azure's **Timeline API** (`/_apis/build/builds/{buildId}/timeline`, which returns a flat list of records, each with `type` (Stage/Job/Phase/Task), `name`, and `result`; when a user specifies `?stage=Deploy`, find the record where `type === 'Stage'` and `name === 'Deploy'`, then read its `result`), or (b) scope them out of this PR and raise it with the maintainer. Current lean: **(a)** to avoid regressing existing behavior, with (b) as a fallback if the maintainer prefers a smaller PR.
5. Add `azure-devops-build.spec.js` — the build badge currently has only a `.tester.js`, no unit spec.

**Implement:** _(Phase III)_ — on branch `fix-issue-10162` in `azizu06/shields`.

**Review:** Self-review against shields' `CONTRIBUTING.md`: PRs are squash-merged (no need to pre-squash my own commits); **service changes must include tests**; the **PR title must tag the affected service in square brackets** so CI runs those service tests — e.g. `[AzureDevops] Use PAT auth for the build badge on private projects`. Commit messages/authorship are not amendable after merge.

**Evaluate:** shields tests two ways and I'll use both — service tests (`azure-devops-build.tester.js`, run with `npm run test:services -- --only=azure-devops`, must keep ≥1 live "picture check") and unit tests (`*.spec.js` using `nock` to fake Azure). Specifically I'll: (1) add a `nock` unit test asserting `Authorization: Basic …` **is** sent when a token is configured (the inverse of the reproduction); (2) add unit tests mapping Azure `result` values (`succeeded` / `failed` / `canceled` / `partiallySucceeded`) to badge output; (3) keep a live picture-check test green; (4) run `npm test` to confirm no regressions. **No real Azure account needed** — `nock` supplies the fake responses.

**Implementation plan commit:** [`c35f39f`](https://github.com/azizu06/shields/commit/c35f39fb60) — adds [`codepath/plan.md`](https://github.com/azizu06/shields/blob/issue-10162-docs/codepath/plan.md)

### Phase III — Solution Building

> **Completion signal:** WIP branch with active daily commits

#### WIP Branch

- **Branch URL:** https://github.com/azizu06/shields/tree/fix-issue-10162
- **First commit date:** 2026-06-12
- **Most recent commit date:** 2026-06-28

#### Implementation Notes

Built in two reviewable increments so the core fix stands on its own and the larger stage/job work is a separate, revertable commit.

| Date | Note |
|---|---|
| 2026-06-21 | **Increment 1 — the auth fix** (commit [`75c5f41`](https://github.com/azizu06/shields/commit/75c5f41d67)). The whole bug is one line: changed `AzureDevOpsBuild` to `extends AzureDevOpsBase` (was `BaseSvgScrapingService`). That base class carries `static auth = { passKey: 'azure_devops_token' }` and runs every request through `withBasicAuth(...)`, so the PAT is now attached. Rewrote `handle()` to call the authenticated `/_apis/build/builds` JSON endpoint, read `value[0].result`, and translate Azure's result vocabulary into the existing `renderBuildStatusBadge` output. Added `azure-devops-build.spec.js` (the badge had no unit spec at all) asserting the PAT is sent — the direct inverse of the Phase II reproduction. |
| 2026-06-21 | **Test parity cleanup.** Two old live tests (`unknown definition`, `unknown project`) were written for the old status-*image* endpoint. The JSON API redirects inaccessible resources to a sign-in page (verified: HTTP 302), so it genuinely cannot tell those two cases apart. Replaced them with a deterministic nock-mocked 404 test and a consolidated `user or project not found` message — exactly how the sibling coverage/tests badges are already tested. |
| 2026-06-21 | **Increment 2 — stage/job parity** (commit [`094a156`](https://github.com/azizu06/shields/commit/094a156d00)). The builds-list call returns whole-build status only, so to keep `?stage=`/`?job=` working I added a call to Azure's **Timeline API** (`/_apis/build/builds/{id}/timeline`) that finds the matching `Stage`/`Job` record and reads its own `result`. Confirmed two non-obvious facts against the live API before coding: the Timeline endpoint **rejects** the `api-version` the other endpoints require (404 with it, 200 without), and it uses different result words (`succeededWithIssues`, `skipped`) than the build-level call — both handled in the result map. |
| 2026-06-28 | **Review feedback round** (commit [`509bc0f`](https://github.com/azizu06/shields/commit/509bc0fd0b)). Per the maintainer's code review, aligned the not-found message with the sibling badges: switched to `build pipeline not found` and treated a pipeline with no completed builds the same way (dropped the earlier `never built` text). Replaced the mocked 404 test with live not-found tests against the public `swellaby` org, matching how the coverage/tests testers already do it. |

**Key challenge & how I solved it:** perfect error-message parity turned out to be impossible, and finding that out drove the design. The old badge scraped an anonymous status *image* that embedded text like "set up now"; the authenticated JSON API has no equivalent and bounces unauthenticated/unknown requests to a login page. So I worked empirically — `curl`'d the real Azure endpoints to see the actual responses (302 redirects, BOM-prefixed JSON, the api-version quirk) and let that evidence shape both the code and the tests, rather than guessing the API's behavior.

#### Testing Strategy

Wrote tests alongside the code, using both layers shields supports.

- **Unit spec (`azure-devops-build.spec.js`, new):** `testAuth` asserts the badge sends `Authorization: Basic …` when a token is configured — the inverse of the Phase II reproduction. (Exercises the overall-build path so a single mocked request suffices.)
- **Service tests (`azure-devops-build.tester.js`, updated):** kept the live "picture-check" cases green against the public `totodem/shields.io` project, with live not-found tests against `swellaby` and **deterministic nock-mocked** cases that pin the new logic:
  - overall build `succeeded` but the requested stage `failed` → badge must show **`failing`** (proves the Timeline lookup is used, not the whole-build result);
  - a job returning `succeededWithIssues` → **`passing` / orange** (job precedence + timeline-specific mapping);
  - a nonexistent stage → **`stage not found`**;
  - an unknown pipeline / bad project → **`build pipeline not found`** (live).
- **Result:** 1 unit spec + service-test cases pass; the sibling Azure badges' specs still pass (no regressions). No real Azure account needed — `nock` supplies the responses.
  - Run: `npx cross-env NODE_CONFIG_ENV=test mocha core/service-test-runner/cli.js --only=AzureDevopsBuild` and `… mocha "services/azure-devops/*.spec.js"`.

**Test file(s):** `services/azure-devops/azure-devops-build.spec.js` (new), `services/azure-devops/azure-devops-build.tester.js` (updated)

#### Mentor Feedback Requests

_None requested this phase. Raised the not-found-message consolidation with the maintainer in the Phase IV PR description instead._

| Date | Question / request | Response |
|---|---|---|
| | | |

### Phase IV — Pull Request & Submission

> **Completion signal:** Submitted PR + evidence of maintainer communication

#### Pull Request

- **PR URL:** https://github.com/badges/shields/pull/11945
- **PR title:** [AzureDevops] Make the build badge send the PAT so it works on private projects
- **Submitted date:** 2026-06-21
- **Status:** Merged

#### PR Description Summary

This PR fixes #10162, where the Azure DevOps build badge ignored the configured PAT and broke on private projects. The build badge was the only Azure badge still scraping the anonymous status image instead of hitting the authenticated JSON API, so it never sent the token. I moved it onto `AzureDevOpsBase` like the coverage, tests, and release badges already do, so it reads the build result from `/_apis/build/builds` with the PAT attached, and kept the `stage`/`job` support working through Azure's Timeline API. I tested it with a new auth unit spec that proves the token is actually sent, plus service-test cases (live and nock-mocked) covering the overall build, stage, job, and not-found paths. One behavior change I flagged in the PR: the JSON API can't tell "definition not found" apart from "user or project not found" the way the old image endpoint could, so I consolidated those two messages to match how the sibling Azure badges already behave.

#### Pre-submission Checklist

- [x] Code addresses the selected issue
- [x] Tests written and passing
- [x] Documentation follows the project's contribution guidelines
- [x] PR description explains the change clearly
- [x] Self-reviewed against the project's CONTRIBUTING.md

#### Maintainer Feedback Log

_Log every round of review feedback and your response. This is evidence of professional iteration._

| Date | Reviewer | Feedback | My response / commit |
|---|---|---|---|
| 2026-06-21 | PyvesB (maintainer) | The temp AI planning files shouldn't be in the PR. | Rebased the 3 planning commits off `fix-issue-10162` so the PR is only the three `azure-devops` source/test files; preserved the planning docs on a separate `issue-10162-docs` branch; replied on the thread. |
| 2026-06-28 | PyvesB (maintainer) | Code review: use `build pipeline not found` to match the `AzureDevOpsCoverage`/`AzureDevOpsTests` badges; keep live not-found tests instead of the mocked 404; asked whether the stage/job tests could be live too. | Changed the message (and the no-completed-builds case) to `build pipeline not found`; replaced the mocked 404 with live tests against `swellaby` like the sibling testers; left the specific-outcome stage/job tests mocked and explained that they can't be reproduced against a public project. Commit `509bc0fd0b`; replied on the thread. |
| 2026-06-28 | PyvesB (maintainer) | **Approved** the PR: "Looks good to me, thanks for the contribution!" No further changes requested. | No code changes needed. |
| 2026-06-28 | PyvesB (maintainer) | **Merged** the PR into `master` (merge commit `2b58f5b`). | Contribution cycle complete. |

---

## Contribution Cycle Log

_If you complete a full cycle and start a second one, add a new section above and archive the previous cycle here with a summary._

| Cycle | Issue | PR | Outcome |
|---|---|---|---|
| 1 | [#10162](https://github.com/badges/shields/issues/10162) | [#11945](https://github.com/badges/shields/pull/11945) | **Merged** into `master` (2026-06-28) — review feedback addressed, CI green |
| 2 | [#11286](https://github.com/badges/shields/issues/11286) | _pending_ | In progress — Phase I (claimed; scope reshaped by maintainer into a `GitHubIssues` / `GitHubPullRequests` badge split) |
