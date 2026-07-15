# Contribution 2: docs(azd): document well-known runtime hosts that `azd` consumes

**Contribution Number:** 2  
**Student:** Dipesh Pandit  
**Issue:** https://github.com/Azure/azure-dev/issues/4415  
**Pull Request:** https://github.com/Azure/azure-dev/pull/9165  
**Status:** Phase IV — PR submitted, awaiting maintainer review

---

## Problem Summary

The Azure Developer CLI (`azd`) reaches out to a number of external services at runtime — ARM management APIs, telemetry, and experimentation endpoints — but there is no single documented list of the hosts it needs. This matters for organizations running `azd` in locked-down networks behind firewalls or traffic-filtering appliances, who currently have to reverse-engineer the allowlist by trial and error. The issue asks a contributor to catalog these well-known hosts and add clear documentation for them. I chose it because it is a small, well-scoped `good first issue` / `help wanted` task with a clear definition of "done," making it a realistic first open-source contribution.

---

## Why I Chose This Issue

As an IT intern, I use Azure to implement AI solutions for process automation. This issue aligns directly with my current professional work, making it a highly relevant contribution — I regularly work inside the Azure ecosystem, so improving the documentation around `azd` is something that has immediate value both to me and to teams like mine.

Beyond the personal relevance, it is well-scoped for a first contribution and has a clear finish line. It is labeled `good first issue` and `help wanted`, is open and unassigned, and asks for a bounded deliverable — a documented list of the external hosts `azd` requires — rather than an open-ended refactor. That makes it something I can fully understand, research, and complete within the program timeline.

It also matches my learning goals. Producing this documentation means reading through the `azd` codebase to trace where network calls are made (ARM, telemetry, and experimentation endpoints), which is a great way to learn how a real production CLI is structured while making a contribution that has concrete value for teams deploying `azd` in restricted network environments. The Azure `azure-dev` repository is actively maintained and has clear contribution guidelines, which gives me a supportive environment to learn enterprise engineering standards.

---

## Understanding the Issue

### Problem Description

This is a **documentation gap**, not a code defect. `azd` connects to many external
hosts at runtime — the Azure ARM control plane, Entra sign-in, Microsoft Graph,
telemetry, a feature-experimentation service, tool-download sources, template and
extension registries, and self-update endpoints. These hosts are hardcoded across
many different Go source files, but there is **no single page** in the
documentation that lists them. Anyone running `azd` behind a firewall or proxy has
to reverse-engineer the allowlist by reading the source or watching network traffic.

### Expected Behavior

There should be one reference doc (in `docs/reference/`) that lists every
well-known host `azd` may contact, grouped by category (control plane per Azure
cloud, auth, telemetry, experimentation, tools, templates, extensions, self-update),
with a copy-pasteable minimum allowlist for firewall configuration.

### Current Behavior

No such page exists. Searching `docs/` for "firewall", "allowlist", "endpoint", or
"host" returns nothing that catalogs the hosts. The endpoint constants live only in
the source (e.g. `cli/azd/pkg/cloud/cloud.go`, `internal/telemetry/telemetry.go`,
`cmd/middleware/experimentation.go`), scattered across ~20 files.

### Affected Components

Documentation only — the fix adds a new Markdown file under `docs/reference/` and
links it from `docs/README.md`. No product code changes. The source files that
*define* the hosts are the research inputs, not files to be modified:

- `cli/azd/pkg/cloud/cloud.go` — per-cloud ARM / portal / storage / ACR / Key Vault suffixes
- `cli/azd/pkg/graphsdk/graphsdk.go` — Microsoft Graph host
- `cli/azd/internal/telemetry/telemetry.go` — Application Insights endpoints
- `cli/azd/cmd/middleware/experimentation.go` — experimentation (TAS) endpoint
- `cli/azd/pkg/tools/{bicep,github,pack}/…`, `pkg/project/container_helper.go` — tool + builder-image download hosts
- `cli/azd/pkg/templates/…`, `pkg/extensions/manager.go`, `pkg/update/manager.go` — template, extension, and self-update hosts

---

## Reproduction Process

### Environment Setup

- **OS:** macOS (darwin), zsh.
- **Clone:** `git clone https://github.com/Azure/azure-dev.git` — cloned cleanly.
- **Note on the toolchain:** the CLI is written in Go (`cli/azd/`). Go was not
  installed on my machine, so I could not build/run the `azd` binary. For a
  **documentation** issue this is not a blocker — reproducing the gap only
  requires reading and searching the repository, not compiling it. (If I later
  want to build the CLI to verify links, I would install Go via `brew install go`
  and run `cd cli/azd && go build ./...`.)
- **No other setup friction.** No dependency install, `.env`, or dev container was
  needed to inspect the docs and source.

### Steps to Reproduce

Because this is a missing-documentation issue, "reproducing" it means confirming
that (a) the docs contain no consolidated host list, and (b) the hosts really are
scattered across the source. Anyone can follow these steps:

1. Clone the repo and enter it:
   ```bash
   git clone https://github.com/Azure/azure-dev.git
   cd azure-dev
   ```
2. Look for any existing firewall / allowlist / endpoints reference in the docs:
   ```bash
   ls docs/reference/
   grep -ri "firewall\|allowlist\|allow-list\|endpoints to\|hosts to" docs/
   ```
   **Result:** `docs/reference/` contains `azure-yaml-schema.md`,
   `environment-variables.md`, `feature-status.md`, and `telemetry-data.md` — but
   **no** page that lists the hosts `azd` contacts. The grep returns no such catalog.
3. Confirm the hosts exist and are spread across the source:
   ```bash
   grep -rn "management.azure.com\|login.microsoftonline\|applicationinsights.azure.com\|exp-tas.com\|downloads.bicep.azure.com" cli/azd --include=*.go | grep -v _test
   ```
   **Result:** matches appear in many different files (cloud config, telemetry,
   experimentation middleware, tool downloaders, etc.), confirming there is no
   single source of truth a firewall admin could consult.

**Expected:** a documented, consolidated list of runtime hosts.
**Actual:** no such document exists; the information is only discoverable by
reading source across ~20 files.

I ran the searches more than once against a fresh clone to confirm the result is
consistent, not a fluke.

### Reproduction Evidence

- **Working branch (fork):** `fix-issue-4415` →
  https://github.com/dipeshpandit12/azure-dev/tree/fix-issue-4415
  (fork created under `dipeshpandit12`, branch pushed — the link resolves). The
  branch is currently clean (based on `main`); the documentation itself is written
  in Phase III per the assignment ("no finished code" in Phase II).
- **My findings:** I traced every runtime host to its defining file and line. The
  hosts fall into clear categories — per-cloud ARM/auth/portal/storage/ACR/Key Vault
  suffixes (`pkg/cloud/cloud.go`), Microsoft Graph (hardcoded to *public* cloud even
  for Gov/China clouds — a notable gotcha), Application Insights telemetry
  (`internal/telemetry/telemetry.go`), the experimentation TAS service
  (`cmd/middleware/experimentation.go`), tool downloads (Bicep, `gh`, `pack`, and the
  `mcr.microsoft.com` builder image), template/extension registries (via `aka.ms`),
  and self-update endpoints. This confirms the issue is valid and gives me the exact
  content the new doc needs to contain.

---

## Solution Approach

### Analysis

The "root cause" here is simply that the documentation was never written. The
information exists and is stable — the hosts are hardcoded constants — but it has
never been collected into a user-facing reference. So the fix is a research +
writing task: read the source, catalog the hosts by category, and publish a single
reference page in the format the repo already uses for its other reference docs.

### Proposed Solution

Add a new file, `docs/reference/network-endpoints.md`, that documents every
well-known runtime host grouped by category, with per-cloud tables (AzureCloud,
AzureUSGovernment, AzureChinaCloud) and a copy-pasteable minimum allowlist. Link
it from the Reference section of `docs/README.md` so it is discoverable.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** `azd` contacts many external hosts at runtime, but there is no
single documented list. Users in restricted networks need one place to find the
hosts to allowlist. The deliverable is documentation, not code.

**Match:** The repo already has a strong pattern for reference docs. I'll mirror
`docs/reference/telemetry-data.md` and `docs/reference/environment-variables.md`
— same tone, Markdown table style, GitHub `[!NOTE]`/`[!IMPORTANT]` callouts, and
relative links back into the source. New reference docs are indexed under the
"Reference" heading of `docs/README.md`, and the "Where do new docs go?" table in
that README confirms configuration references belong in `docs/reference/`.

**Plan:**
1. Add `docs/reference/network-endpoints.md` with sections for: Azure control
   plane (per cloud), authentication, telemetry, experimentation, external-tool
   downloads, templates, extensions, self-update, `aka.ms` redirector, CI/CD
   provider hosts, and customer/deployment-target hosts.
2. Include a per-cloud table (public / US Gov / China) and a copy-pasteable
   "minimum allowlist" block for a typical `provision` + `deploy`.
3. Cite the defining source file for each host so the doc stays verifiable.
4. Add one link to the new page under the "Reference" section of `docs/README.md`.

**Implement:** *(Phase III)* — branch `fix-issue-4415` in my fork:
`https://github.com/dipeshpandit12/azure-dev/tree/fix-issue-4415`

**Review:** Read `CONTRIBUTING.md` and `docs/README.md` conventions before opening
the PR; match the existing reference-doc style; confirm all relative links resolve;
follow the repo's commit-message convention (`docs(azd): …`). Since this is
docs-only, I'll confirm there are no product-code changes in the diff.

**Evaluate:** Verify every source-file link in the doc resolves to a real path;
re-run the reproduction greps to confirm no host was missed; optionally build the
CLI (`go build ./...`) only if I need to double-check a constant's value. Success =
a reader can configure a firewall from this one page without reading source.

---

## Testing Strategy

> This is a documentation-only change, so automated unit/integration tests do not
> apply. "Testing" means verifying the doc is accurate and well-formed.

### Verification checklist (Phase III) — completed

- [x] Every source-file link in the new doc resolves to a real path in the repo
- [x] Re-run the reproduction greps to confirm no runtime host was missed
- [x] Markdown renders correctly on GitHub (tables, callouts, code blocks)
- [x] Relative links from `docs/reference/` back into `cli/azd/…` are correct
- [x] `docs/README.md` links to the new page under "Reference"
- [x] Spell-check (`cspell`) passes — new host-name terms added to `.vscode/cspell.misc.yaml` so CI does not flag them

### Manual Testing

Spot-check a few host values against the source (e.g. confirm
`cli/azd/pkg/cloud/cloud.go` and `internal/telemetry/telemetry.go` still contain
the documented endpoints) so the doc matches the current code.

---

## Implementation Notes (Phase III — Build)

### What I built

I wrote a single new reference page, `docs/reference/network-endpoints.md`
(~270 lines), that catalogs every well-known host `azd` may contact at runtime.
The page is organized exactly the way I planned it in Phase II:

- **Section 1 — Azure control plane (per cloud):** three tables (AzureCloud,
  AzureUSGovernment, AzureChinaCloud) covering ARM, the Entra/AAD authority,
  Microsoft Graph, the portal, and the storage / ACR / Key Vault suffixes.
- **Sections for auth, telemetry, experimentation, external-tool downloads
  (Bicep, `gh`, `pack`, the `mcr.microsoft.com` builder image), templates,
  extensions, self-update, and CI/CD provider hosts.**
- **A copy-pasteable "minimum allowlist"** for a typical `provision` + `deploy`
  flow, which is the payoff a firewall admin actually needs.
- Each host cites the **defining source file** (e.g.
  `cli/azd/pkg/cloud/cloud.go`, `internal/telemetry/telemetry.go`) via a relative
  link, so the doc stays verifiable against the code.

I matched the existing reference-doc conventions in the repo — the same table
style, GitHub `[!NOTE]`/`[!IMPORTANT]` callouts, and relative source links used
in `telemetry-data.md` and `environment-variables.md`.

### Challenges / decisions

- **cspell CI gate.** After my first local render I discovered the repo runs a
  spell-checker in CI. The new host names (`usgovcloudapi`, `chinacloudapi`,
  `azurecr`, etc.) tripped it. Rather than litter the doc with inline ignore
  comments, I added the terms to the shared dictionary at
  `.vscode/cspell.misc.yaml`, which is the pattern the project already uses.
- **Microsoft Graph gotcha.** While tracing the source I confirmed the Graph host
  is hardcoded to the *public* cloud (`graph.microsoft.com`) even for Gov/China
  clouds. I called this out explicitly in the doc as a note so a Gov/China admin
  isn't surprised.
- **Runtime-constructed hosts.** Some hosts (storage, ACR, Key Vault) are suffixes
  assembled at runtime from resource names. I documented these with a `<...>`
  placeholder and a wildcard (e.g. `*.azurecr.io`) rather than pretending they are
  fixed hostnames.
- **No Go build needed.** As planned, this is a docs-only change, so I verified
  host values by reading the source constants rather than compiling the CLI.

### Version-control hygiene

I worked on the `fix-issue-4415` branch off `main`, kept the change scoped to
docs + the cspell dictionary (no product code touched), and used a single
Conventional-Commits message with a `Fixes #4415` trailer so the PR auto-links
and closes the issue on merge.

### Code Changes

- **Files added/modified:**
  - `docs/reference/network-endpoints.md` — **new** (270 lines) — the reference page.
  - `docs/README.md` — +1 line — link to the new page under the "Reference" heading.
  - `.vscode/cspell.misc.yaml` — +7 lines — host-name terms so CI spell-check passes.
- **Key commit:** `d687faaac` — `docs: document runtime network endpoints azd contacts (#4415)`
  (branch `fix-issue-4415`, pushed to my fork `dipeshpandit12/azure-dev`).
- **Approach decisions:** mirror the existing reference-doc format instead of
  inventing a new layout; cite source files for every host so the page is
  verifiable; solve the cspell failure via the shared dictionary rather than
  per-line suppressions.

---

## Pull Request (Phase IV — Submit & Iterate)

**PR Link:** https://github.com/Azure/azure-dev/pull/9165

I opened the pull request against `Azure/azure-dev:main` from
`dipeshpandit12/azure-dev:fix-issue-4415`. Before submitting I re-read
`CONTRIBUTING.md` and the `docs/README.md` conventions, confirmed the diff
contains no product-code changes, and verified every relative link resolves.

**PR Description (as submitted):**

> **docs: document runtime network endpoints `azd` contacts**
>
> Fixes #4415.
>
> Adds `docs/reference/network-endpoints.md`, a single reference page cataloguing
> the external hosts `azd` may contact at runtime — Azure control plane (per
> cloud), authentication, telemetry, experimentation, tool downloads, templates,
> extensions, self-update, and CI/CD provider hosts — plus a copy-pasteable
> minimum allowlist for firewalled environments. Every host cites the source file
> that defines it. The page is linked from the Reference section of
> `docs/README.md`, and new host-name terms are added to the shared cspell
> dictionary so CI spell-check passes.
>
> This is a documentation-only change; no product code is modified.

**Maintainer Feedback:**
- _2026-07-15:_ PR submitted; automated checks (including the cspell CI gate) run
  on push. Awaiting maintainer review — no human feedback received yet.
- _(This section will be updated as review comments come in and I push revisions.)_

**Status:** Awaiting review

---

## Learnings & Reflections

### Technical Skills Gained

- **Reading an unfamiliar production codebase to trace behavior.** I learned to
  follow `azd`'s network calls from feature code down to the constants that define
  each host, across ~20 Go files, without needing to run the binary.
- **Matching a project's conventions.** I studied the existing reference docs and
  reproduced their table style, callout syntax, and source-linking pattern instead
  of imposing my own format — the single most useful habit for getting a PR merged.
- **CI awareness.** I learned that "docs-only" still has to pass automated gates.
  Discovering and satisfying the cspell dictionary was a small but real lesson in
  reading a project's CI before assuming a change is complete.
- **Git/PR hygiene:** a focused branch, a scoped diff, a Conventional-Commits
  message, and a `Fixes #` trailer that auto-links the issue.

### Challenges Overcome

- **No Go toolchain locally.** I couldn't build the CLI, so I had to be confident
  the doc was correct purely from reading source. I handled this by citing the
  defining file for every host and re-running my reproduction greps to confirm I
  hadn't missed one.
- **The hidden cspell failure.** The change looked done, but CI would have failed
  on unknown host names. Tracking that down and fixing it the project's way (shared
  dictionary, not inline suppressions) was the main non-obvious hurdle.
- **Deciding what to document vs. omit.** Runtime-constructed hosts and the
  public-cloud Graph gotcha forced judgment calls about accuracy vs. clutter; I
  resolved them with wildcards/placeholders and explicit notes.

### What I'd Do Differently Next Time

- **Check CI configuration first.** I'd read `.github/workflows` and the linting
  config *before* writing, so I don't discover a gate like cspell after the fact.
- **Comment on the issue before starting.** For a `good first issue`, a quick note
  to the maintainer confirming the scope and proposed file location would reduce
  the risk of a rework request during review.

---

## Resources Used

- Azure `azure-dev` repository — `CONTRIBUTING.md` and `docs/README.md` conventions
- Existing reference docs I used as templates: `docs/reference/telemetry-data.md`,
  `docs/reference/environment-variables.md`
- Source files traced for host constants: `cli/azd/pkg/cloud/cloud.go`,
  `cli/azd/internal/telemetry/telemetry.go`,
  `cli/azd/cmd/middleware/experimentation.go`, and the tool/template/extension
  packages under `cli/azd/pkg/…`
- GitHub docs on [Conventional Commits](https://www.conventionalcommits.org/) and
  linking a PR to an issue with `Fixes #`
- [cspell](https://cspell.org/) configuration reference (for `.vscode/cspell.misc.yaml`)
- Original issue: https://github.com/Azure/azure-dev/issues/4415
