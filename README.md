# Provider Gap Matrix

## Goal

Map current provider coverage, identify missing deployment models, and clarify where Shipper appears narrower than it should.

## Current providers

This is a capability inventory, not proof of a successful live deployment. Read
each provider's manifest and limitations before choosing it; pending audit PRs
do not establish behavior in an installed release.

| Provider | Category | Primary model |
| --- | --- | --- |
| Ploi | Server panel | Managed VPS / server deployments |
| Laravel Forge | Server panel | Laravel-oriented server deployments |
| EasyPanel | App/server panel | Managed application and service deployments |
| cPanel | Shared hosting panel | Shared hosting deployments |

Coolify also has an implementation in
[shippercli/provider-coolify](https://github.com/shippercli/provider-coolify).
As checked on 2026-10-16, its default branch is
`wip/workspace-handoff-2026-10-08` and the repository has no version tags.
Its application, service, database, and preview behavior is under audit in
[PR #1](https://github.com/shippercli/provider-coolify/pull/1). Track it as an
implementation awaiting release and lifecycle verification, rather than as an
absent provider or verified production option.

## Coverage by deployment model

| Deployment model | Covered | Providers | Gap severity |
| --- | --- | --- | --- |
| Managed server panel | Yes | Ploi, Forge, EasyPanel | Low |
| Shared hosting panel | Yes | cPanel | Medium |
| Generic SSH / raw server | No | None | High |
| PaaS app platform | No | None | High |
| Frontend / static platform | No | None | High |
| Container-native platform | Partial | EasyPanel; Coolify implementation under audit | High |
| Kubernetes | No | None | High |
| Preview-environment-first platform | No | None | Medium |

## Coverage by capability

Legend:

- `Yes`: implemented by the provider for this feature
- `Partial`: provider-dependent or uneven
- `No`: not covered today

Rows describe provider-specific features; a broad capability declaration does
not imply every subfeature is implemented. In particular, queue workers do not
establish daemon supervision, and deployment history does not establish rollback.
The Ploi rollback and EasyPanel workload entries were checked against their
[Ploi manifest](https://github.com/shippercli/provider-ploi/blob/main/src/PloiProvider.php)
and [EasyPanel manifest](https://github.com/shippercli/provider-easypanel/blob/main/src/EasyPanelProvider.php)
on 2026-10-16; these references describe implementation declarations, not live tests.

| Capability | Ploi | Forge | cPanel | EasyPanel | Overall gap |
| --- | --- | --- | --- | --- | --- |
| App deployment | Yes | Partial | Yes | Yes | Medium |
| Domain management | Yes | Yes | Yes | Yes | Low |
| SSL certificates | Yes | Yes | Partial | Partial | Low |
| Database provisioning | Yes | Yes | Yes | Yes | Low |
| Environment variables | Yes | Yes | Yes | Yes | Low |
| Server lifecycle | Partial | No | No | No | High |
| PHP runtime configuration | Yes | No | Yes | No | Medium |
| Custom NGINX configuration | Yes | No | No | No | Medium |
| Queue / worker management | Yes | Yes | No | Yes | Low |
| Daemon / process supervision | Yes | Yes | No | No | Medium |
| Cron / scheduled jobs | Yes | Yes | Yes | Partial | Low |
| App resource limits | No | No | No | No | High |
| Service mounts | No | No | No | No | High |
| Preview environments | Partial | Partial | Partial | Partial | Medium |
| Multi-service apps | No | No | No | Partial | High |
| Container image deploys | No | No | No | Yes | High |
| Rollbacks | No | No | Partial | No | High |
| Logs / runtime status | Partial | Yes | Partial | Partial | Medium |

### EasyPanel scope notes

The [provider's version 1 scope](https://github.com/shippercli/provider-easypanel/blob/main/README.md)
and implementation distinguish panel features from features this provider exposes:

- HTTPS domain bindings delegate certificate issuance and renewal to EasyPanel;
  the provider does not manage certificate resources directly.
- Scheduled jobs run as scripts in a managed Box service and require a Git or
  GitHub source. Image-only application sources do not satisfy that requirement.
- Resource limits, mounts, and persistent-volume lifecycle are explicitly outside
  the provider's supported scope, even when available in the EasyPanel UI.
- Container image sources are implemented directly; that does not imply general
  multi-service orchestration or the unsupported resource controls above.

## Correctness and release evidence

The following is an audit snapshot from 2026-10-16. These are pending changes and
observed checks, not a claim that the fixes are in published releases. Green CI,
readable package metadata, and a declared capability do not prove a deployment
works against a live provider account.

| Area under review | Evidence obtained | Still required |
| --- | --- | --- |
| [CLI PRs #152–#157](https://github.com/shippercli/cli/pull/152) | Combined source built as a production PHAR; startup/help outside checkout and 56 boolean parsing checks pass. A fresh Composer installation discovers and adapts all four provider plugins. | Review and release the compatible changes together; verify provider operations with actual accounts. The PHAR cannot discover separately installed Composer plugins. |
| [Forge #9](https://github.com/shippercli/provider-forge/pull/9) | 60 unit tests / 178 assertions; actual Guzzle redirect/error checks; PHP 8.3–8.5 CI and protected aggregate gate pass; distribution archive checks pass. | Live creation, existing-resource ownership, credential delivery, deployment/runtime, and cleanup verification. |
| [Ploi #26](https://github.com/shippercli/provider-ploi/pull/26) | 79 unit tests / 441 assertions; actual SDK resource routing, token rotation and HTTP rejection checks; PHP 8.3–8.5 CI and distribution checks pass. | Live lifecycle verification; attaching an existing database to a different site still needs a documented Ploi API operation. |
| [EasyPanel #14](https://github.com/shippercli/provider-easypanel/pull/14) | 40 unit tests / 217 assertions; real cURL status/credential-diagnostic checks; PHP 8.3–8.5 CI and distribution checks pass. | Live application/database/workload health, ownership transitions, and cleanup verification. |
| [Coolify #1](https://github.com/shippercli/provider-coolify/pull/1) | 149 unit tests / 842 assertions; three actual HTTP regressions; real CLI configuration objects tested with fake API transport; PHP 8.3–8.5 CI and distribution checks pass. | Unreleased implementation; live authentication, deployment/runtime, previews, mutations and legacy ownership migration remain unverified. |
| [cPanel cleanup #22](https://github.com/shippercli/provider-cpanel/pull/22) and [rollback #21](https://github.com/shippercli/provider-cpanel/pull/21) | Combined audit branches pass 719 tests / 3,327 assertions. Cleanup validates ancestor metadata before trash and inventory after trash. Staged rollback/recovery and packaged helper checks have local regressions. | Shared apply/destroy/rollback locking; concurrent path replacement; live Python/cPanel behavior and account permissions. API observations are not no-follow filesystem handles or a transaction. |
| [Website #11](https://github.com/shippercli/website/pull/11) | Nine tests; webpack production build creates 77 pages; Chrome verifies pause and manual provider advancement. | Browser execution with denied storage/reduced motion, production rendering and content accuracy. Local default Turbopack build hit a worker-port permission restriction. |

### Fresh production installation

A separate Composer consumer installed the four provider ZIPs and the combined
CLI source archive with production dependencies only, using local `dev-audit`
package metadata. On PHP 8.4 with normal ini configuration disabled to exclude
Herd's debug autoloader, all 27 provider runtime classes loaded from installed
packages, plugin capability manifests validated, and the installed CLI's real
registry/factory discovered and adapted every provider. All six provider HTTP
regression scripts passed against this dependency installation. The resulting
84-package graph had no reported vulnerability advisories at observation time.

The tested provider source heads were Forge `6e49422`, Ploi `1868ed7`, EasyPanel
`5f6e2be`, and Coolify `241d5ba`; the local combined CLI head was `5c25c37`.
This verifies packaging/bootstrap compatibility locally on PHP 8.4. It does not
replace deployment checks or establish that a release already contains these
changes. Individual provider CI also checks PHP 8.3 and 8.5; the fresh combined
consumer install has not been repeated on those versions.

### Refreshed consumer verification after Forge and Ploi fixes

On 2026-10-16, a new consumer with no pre-existing lockfile or vendor directory
installed fresh local Forge (`77d7d034c643d0f07a1ee479ff6120463c3d445c`)
and Ploi (`f602b8b3a3f5dfa144e0ca9d442e5956a5bfb921`) Composer ZIPs alongside
the retained CLI, EasyPanel and Coolify archives from the previous consumer.
These remain unreleased audit sources; the retained packages were not rebuilt.

Using PHP 8.4.15, the fresh solve installed 84 production packages with `--no-dev --no-plugins
--no-scripts`, including contracts 1.2.0, Forge SDK 4.1.1, Ploi SDK 1.9.5 and
Guzzle 7.15.5. With host PHP configuration disabled, all 27 provider runtime
classes loaded from installed archives; PHPUnit, Pest, Mockery, Pint and PHPStan
were absent from Composer metadata. The installed CLI discovered/adapted all
four providers, loaded an empty project configuration, and ran version/help
outside its checkout. All six provider HTTP suites passed against this installed
dependency graph, including 213 Forge requests and 150 Ploi requests. The locked
dependency audit reported no advisories.

Lockfile SHA-256:
`dbaa5666760a979522c52e387ba8a58a7c29a676d77804810da98ba05fef0043`.
This is a fresh dependency/install integration check with synthetic local HTTP,
not a live provider deployment, release installation, or proof of all workflows.

### Current provider and combined CLI installation

A subsequent fresh installation on 2026-10-16 rebuilt every Composer ZIP from
these unreleased audit commits:

| Package | Source commit |
| --- | --- |
| Forge | `77d7d034c643d0f07a1ee479ff6120463c3d445c` |
| Ploi | `f602b8b3a3f5dfa144e0ca9d442e5956a5bfb921` |
| EasyPanel | `0db2b80f70cf18d4695f1320731bd3bb8d68788c` |
| Coolify | `5cbe7e2b283d20f70dcb6ad0acaa392f11e2d4fa` |
| CLI, local combination of PRs #152–#158 | `c2e472880da6e65d828f8979239ab4ab9606ee1a` |

The new solve installed 84 production packages without development dependencies,
plugins or scripts. Under PHP 8.4.15 with host configuration disabled, all 27
provider runtime classes loaded from the installed packages; all four plugin
registrations, capability manifests, CLI provider discovery/adaptation, empty
configuration parsing and CLI version startup passed. Composer metadata contained
none of PHPUnit, Pest, Mockery, Pint or PHPStan. The locked dependency audit
reported no advisories or abandoned packages.

All seven current local HTTP suites passed against this installed dependency
graph: Forge responses, Ploi responses, EasyPanel responses, and Coolify error
diagnostics, inventory shapes, database environment delivery and retries.
The harnesses load the installed vendor autoloader and use local HTTP fixtures.
They do not verify live provider accounts, database connectivity or all deployment
workflows. The combined CLI commit is local integration evidence; its PRs remain
unmerged.

Lockfile SHA-256:
`715eb247f661490ba1f1c3f5285555b3298c0a34205d1cde0902518ce6c19555`.

### Organization PR review snapshot, 2026-10-16

A fresh GitHub query on 2026-10-16 inspected all 53 open PRs and their exact
heads. All 148 reported check runs completed successfully; none were pending or
failing. GitHub reported all 53 heads mergeable without conflicts. There were
24 ready PRs and 29 drafts. This is a dated observation, not a guarantee for
later commits.

Twenty-two PRs reported a blocked merge state: cPanel #8–#25 and #27, contracts
#5 and #7, and capability-spec #2. In cPanel, branch protection requires
`CI / PHP 8.3`, `CI / PHP 8.4` and `CI / PHP 8.5`, while the older PR workflows
report `PHP 8.3`, `PHP 8.4` and `PHP 8.5`. The existing one-line
[cPanel #26](https://github.com/shippercli/provider-cpanel/pull/26) reports all
three required contexts successfully and has a clean merge state. The analogous
[contracts #6](https://github.com/shippercli/contracts/pull/6) and
[capability-spec #3](https://github.com/shippercli/provider-capability-spec/pull/3)
corrections are also ready with clean merge states. Review those base workflow
corrections before refreshing dependent PR branches and their checks.

The latest Forge, Ploi, EasyPanel and documentation fixes all have successful
reported CI at this snapshot. Passing CI and conflict-free mergeability do not
establish live lifecycle correctness. Drafts retain their documented deployment,
ownership, credential and concurrency gaps. The author account cannot submit
an approving review on its own PRs; an independent reviewer must provide any
required approval. No PR was merged and no branch protection was changed by
this review.

### Combined cPanel rollback follow-up

After the journal lock-scope fix at rollback PR #21 source
`b46b78e290b1959e651d6dc8ea8acbb2d45f3358`, the local combination of cPanel
PRs #8–#25 at `225d7e9961b49bddfa1f7bbb9024517b675303f6` passed 749 PHP
tests / 3,363 assertions, 27 Python archive tests, all four HTTP suites
(522 cases), and extracted Composer distribution verification (28 files).
The PHP total includes a pre-existing untracked CLI boolean integration test in
the temporary combined checkout; that test is not part of the rollback PR.
Standalone rollback PR CI also passed at the stated source commit.

Recovery and cleanup now require the journal's stage identity, home and active
path to remain bound to the acquired lock. This does not coordinate the remote
rollback process lock with PHP-driven apply/destroy API sequences. Shared
lifecycle locking, concurrent path replacement and live account behavior remain
open. The integration commit is local evidence, not a merged or released package.

### CLI archive correction and clean consumer verification

Earlier absence-of-development-dependency checks inspected the consumer root's
Composer metadata. That did not detect the CLI ZIP's bundled checkout `vendor`
tree. The pre-fix CLI archive contained 7,956 files / 112,788,021 uncompressed
bytes, including PHPUnit, Pest and analysis dependencies. Those earlier checks
prove dependency resolution, not absence of bundled dependency copies.

[CLI #159](https://github.com/shippercli/cli/pull/159) adds archive exclusions and
an archive-content regression. Combined locally with CLI #152–#158 at
`e16ba2b58b643e2de0272fe665d03f201db811ed`, it passes 213 tests / 585 assertions.
The CI merge conflict was resolved locally by retaining both the dependency
audit and archive verification steps. The resulting ZIP contains 151 files /
1,228,535 uncompressed bytes. Direct archive and installed-tree inspection finds
no nested vendor/node_modules, tests, CI or build directories, environment files,
auth.json or box.phar. PR #159 CI passes on PHP 8.3–8.5.

A new consumer installed that archive with the current four provider archives
and 84 production dependencies. Plugin registration, all 27 runtime classes,
CLI discovery/adaptation, empty configuration, version startup outside checkout
and all seven provider HTTP suites passed with host PHP configuration disabled.
CLI archive SHA-256:
`d3d753799d12d2564d1e4ce2ff5709be0c96f08e8453d44386169cc49c2e4aad`.
Consumer lock SHA-256:
`f68c468c1f5f5a32b2b203e57431b597618e899eb65e210940e4b36343c46360`.
This remains unreleased integration evidence and does not verify live deployment.

### Nested package exclusions follow-up

Direct archive-content fixtures exposed nested local files that root-only
exclusions did not remove. The current provider audit heads extend Composer
exclusions to nested vendor/node_modules, environment/auth files and build
outputs. Each fixture failed before the exclusion change and passes afterward;
required source files and extracted runtime classes remain verified. Current
provider PR CI is green at Forge `bf092bc`, Ploi `fdf1b0a`, EasyPanel `959c1a8`
and Coolify `5092466`. These are packaging-only follow-ups to the consumer
verification above, not a new live deployment claim.

[Contracts #7](https://github.com/shippercli/contracts/pull/7) supplies equivalent
exclusions and content checks. Its 18-file archive at `fe08db9` was installed in
a fresh 84-dependency consumer alongside the corrected CLI and four providers.
All 12 contract types, 27 provider runtime classes, registrations, CLI discovery,
startup and seven HTTP suites passed. Installed content was checked directly
for forbidden nested files in all six Shipper packages. The fixture explicitly
aliased the development contracts version to satisfy existing 1.2 constraints;
it does not claim that the fix is published. Contracts #7 CI passes on PHP
8.3–8.5, but the protected check-name mismatch still blocks its merge.

cPanel rollback PR #21 at `5c491d4` also excludes credentials and nested
dependencies. Its expanded package test first reproduced a bundled synthetic
.env file; the corrected 17-file archive still schedules its extracted helper.
The standalone branch passes 27 archive tests and 62 PHP tests / 301 assertions.
Shared lifecycle locking and live account checks remain open.

The three existing one-line CI-name correction PRs (cPanel #26, contracts #6,
capability-spec #3) were rechecked against the actual protected contexts and
all have passing required checks and clean merge states. A user merge decision
has been requested for only these corrections; no default branch or protection
setting has been changed by this review.

### Website preview and credential follow-up

Website PR #11 at `ae76013` passes 42 Node tests, six built-output verifier tests,
TypeScript and the 77-route production build. Checks cover local links across
73 HTML pages and 300-second revalidation for all 17 provider routes. Runtime
metadata shape/slug checks and complete fetch-cache tracking are included.
The exact successful Vercel preview (deployment 7123642288) was fetched through
the connected authenticated Vercel tool: homepage, provider listing/detail,
Forge configuration, database docs and specs returned 200 with expected
server-rendered content; an unknown docs route returned 404. Preview responses
have noindex. Protection settings were not changed. Browser interaction and live
cache expiry remain unverified; unmerged docs changes are not yet published.

Current explicit-password and complete-input fixes are green in CI at Forge
`41ee587` (209 tests / 511 assertions; 225 HTTP requests), Ploi `c71fa32`
(262 / 977; 168 HTTP requests) and EasyPanel `b9960eb` (137 / 448; 241 HTTP
requests). They cover unchanged selected secret bytes, preflight refusal and
complete environment parsing. They do not prove live API provisioning or
database connectivity. Docs PR #4 at `3e573d2` has green content CI; its exact
password example loads through real CLI DTOs and all three selection paths.

### Open correctness issues

- [Credential delivery, contracts #3](https://github.com/shippercli/contracts/issues/3):
  generated connection values must reach the application, with distinct database
  mappings and explicit environment precedence. Unknown existing passwords must
  not be invented or silently rotated. Local/mock evidence exists; live database
  connectivity and migration behavior remain open.
- [Ownership, contracts #4](https://github.com/shippercli/contracts/issues/4):
  exact ownership and complete readable inventories must precede inferred
  mutation/deletion. Response identity checks and refusal paths improve safety;
  concurrent changes, legacy migration, and live lifecycle checks remain open.
- [Existing Ploi database association, CLI #134](https://github.com/shippercli/cli/issues/134):
  refusing a database attached to another site avoids false success but does not
  satisfy the requested attach-existing acceptance criterion. Keep the issue open
  until a documented operation can be implemented and tested.

Before treating an audited provider as release-ready, verify its actual
create/reapply/deploy/runtime/status/logs/cleanup flow, existing-resource refusal
and adoption boundaries, generated database connectivity, failed/partial
operations, and supported preview/rollback behavior. Complete reviews and
release integration remain necessary even when local tests and CI pass.

## Provider follow-up snapshot — 2026-10-16

This snapshot supplements the earlier audit evidence; these changes remain in
open PRs and are not evidence of a released or live-tested deployment.

| Provider PR | Reviewed head | Follow-up evidence |
| --- | --- | --- |
| [Forge #9](https://github.com/shippercli/provider-forge/pull/9) | `c92544fa303a4000a5685eed9f70c1106bae10f2` | 228 tests / 575 assertions; 247 local HTTP requests; current-head CI successful. Validates the actual server database engine against declarations and delivers confirmed connections before later database creation. |
| [Ploi #26](https://github.com/shippercli/provider-ploi/pull/26) | `5674a0e5da8241285d6b3b5656dcf0dd7c1a9822` | 275 tests / 1,047 assertions; 192 local HTTP requests; current-head CI successful. Each confirmed connection is merged into the site environment and read back before provisioning continues. A stateful SDK fixture verifies fresh retry creates only the missing database and retains both passwords and unrelated environment content. |
| [EasyPanel #14](https://github.com/shippercli/provider-easypanel/pull/14) | `71e9130e21c80df522d5aa2d78a7798d2383b6d2` | 137 tests / 449 assertions; 241 local HTTP requests; current-head CI successful. Corrected four test assertion scopes that could accidentally catch their own failures; a disabled source guard reproduced the previous false pass. |
| [Coolify #1](https://github.com/shippercli/provider-coolify/pull/1) | `5092466077004c0157df829172670c96c9ed6571` | Current-head CI successful. Packaging exclusions are verified at nested paths; earlier runtime evidence remains scoped to its recorded source heads. |

Forge's engine check follows the official create-database/server contract; it
does not send an invented per-schema engine field. Ploi rejects unsupported
non-SQL declaration types locally, but actual existing-server engine matching
remains unverified. Accepted labels alone do not prove server compatibility.

Ploi's partial-provisioning regression failed before the delivery fix. Its fresh
retry fixture checks the sequence `create:main`, delivery, failed
`create:analytics`, retry `create:analytics`, delivery. This is a simulated API
through the real SDK, not live database connectivity. Failed or ambiguous
creation/storage still requires durable recovery; neither provider has an
atomic multi-database transaction or durable credential journal.

The refreshed organization inventory contains **50 open PRs** (20 ready for
review, 30 draft) and the same three open issues: contracts #3, contracts #4,
and CLI #134. No failed checks were reported in that snapshot. Check success
alone does not approve a PR or establish release readiness. Protected-branch
check-name corrections still await the requested merge decision.

## Refreshed production consumer — 2026-10-16

A fresh isolated Composer consumer installed six audited archives with
`--no-dev --no-plugins --no-scripts` and 84 production dependencies. Contracts
uses `dev-audit as 1.2.0` solely as a compatibility fixture, not a published release.

| Package | Source revision |
| --- | --- |
| provider-forge | `0edfeca779f95212d43f0e72e5d31f7e419ff88f` |
| provider-ploi | `ed881f4a7df801ba001f53cd0d43a00f55c54ab8` |
| provider-easypanel | `71e9130e21c80df522d5aa2d78a7798d2383b6d2` |
| provider-coolify | `5092466077004c0157df829172670c96c9ed6571` |
| cli | `e16ba2b58b643e2de0272fe665d03f201db811ed` |
| contracts | `fe08db9199dd473d8de494dfa230fa53fc8bb446` |

Installed-package checks passed with PHP `-n`: 27 provider runtime classes,
12 contract types, four provider registrations/manifests, CLI discovery/adapters,
empty-project configuration loading and version startup through Composer's
`vendor/bin/shipper` proxy. Direct invocation of the nested package executable
cannot locate the consumer autoloader; the supported Composer proxy succeeds.
All seven HTTP fixture suites passed against installed dependencies, including
Forge persistence/raw-envelope checks and Ploi partial retry/final readback.
Fixtures were verified against their recorded Git heads and kept outside packages.

All six ZIPs were inspected for bundled dependency, development, local-secret
and build content; no forbidden paths were present. Archives were built from Git
exports followed by Composer archive; separate package verifiers cover nested
source-copy exclusion behavior. The combined CLI revision is a local integration
of the open CLI PRs, not a GitHub merge. Consumer lock SHA-256:
`bba407215464c177dfd96e1de9dc2f7785f54468a395ef8b625e59bc0e5b5a82`.

This verifies the recorded fixtures run from production-installed packages.
Live deployments, database connectivity, persistence timing, durable credential
recovery, cPanel lifecycle concurrency and published-package availability remain
unverified. No PR was merged for this integration check.

## Website browser verification — 2026-10-16

Website source `ae760136d22dc3a0402697f78427a8c4acc9452e`, existing local
production build `e1TFLfcLigvXLHTwKNe6b`, was exercised in installed headless
Google Chrome using bundled Playwright and a local production server. A storage
getter throwing `SecurityError` was installed before page scripts.

Observed assertions passed: dark system preference fallback, interactive theme
toggle despite denied storage, reload returning to the system preference,
reduced-motion suppression of automatic provider rotation, manual Next provider,
automatic rotation after changing to no-preference, and stopped rotation after
changing back to reduce. No browser page-error events were reported. The static
observations spanned more than the two-second rotation interval.

This verifies the existing local build's client behavior. It is not a fresh build,
a hosted-preview browser run, a full accessibility audit or live CDN revalidation
evidence. Hosted-preview HTTP checks remain separate. The evidence is also
recorded on [website PR #11](https://github.com/shippercli/website/pull/11).

## Current review disposition — 2026-10-16

The refreshed organization inventory contains 51 open PRs. After completing the
documentation readiness review, 22 are ready for review and 29 remain draft.
No failed checks were reported in the inventory snapshot. The three open issues
remain contracts #3 (credential delivery), contracts #4 (ownership) and CLI #134
(existing Ploi database association).

- CLI #152–159 remain ready, with combined local integration and production-package
  evidence recorded above. This does not merge the branches or publish a release.
- Website #11 is ready at `8e55b000c7f2f362202c466b0d32808bb0ec225b`;
  current-head CI/Vercel checks pass. Mobile browser regressions exposed and then
  verified fixes for overflowing cards and duplicate index titles. Fresh local
  production browser checks passed at widths 320 and 390.
- Docs #4 is now ready at `a6febdbfa57bbffc9fee48b706a300807ad0ecdb` after
  source validation, exact YAML/provider-selection checks and candidate rendering.
  Its readiness applies to documentation, not live provider acceptance.
- Specs #6 is ready and adds readable source summaries plus Ploi, deploy-command
  and test-coverage source reconciliation. Current-head content CI passes at
  `547303cd` (abbreviated revision); historical bodies elsewhere still need review.
- cPanel #26, contracts #6 and capability-spec #3 are reviewed CI-name corrections
  awaiting the previously requested merge decision. Other ready PRs blocked by
  those required context names must not be described as failing their tests.
- Provider implementation drafts remain draft because full live lifecycle,
  credential connectivity/recovery, legacy migration and concurrency acceptance
  is incomplete. The shared contracts/capability proposals remain open drafts.

Ready status means a change is reviewable; it is not an approval, merge, release,
or proof that the whole audit is complete. No PR was merged during this review.

## Deployment follow-up production integration — 2026-10-16

A new production-only consumer verified these source revisions together:

| Package | Source revision |
| --- | --- |
| provider-forge | `3afcd4fc1a162c09eadcf33d4ae19d9dfd1fd6c8` |
| provider-ploi | `059b6c72d4e9af76a33cfbb7a496947f95394143` |
| provider-easypanel | `5c6e08cdf67962d7a8d7fbd7ddb45591ea837c42` |
| provider-coolify | `5092466077004c0157df829172670c96c9ed6571` |
| cli | `e16ba2b58b643e2de0272fe665d03f201db811ed` |
| contracts | `fe08db9199dd473d8de494dfa230fa53fc8bb446` |

All seven SDK HTTP suites passed using installed package code/dependencies, with
278 Forge requests, 214 Ploi requests and 244 EasyPanel requests plus all four
Coolify suites. The tested Forge client now waits for its exact created deployment
ID to finish; Ploi validates trigger acknowledgments, site identity/states and
monotonic deadlines. Ploi unique current-deployment correlation remains open.
EasyPanel rejects incomplete ownership evidence through serialized cleanup paths.

Installation used 84 production dependencies and `--no-dev --no-plugins
--no-scripts`. The contracts version alias is only a compatibility fixture. Six
archives passed forbidden-content inspection. PHP `-n` verified 27 provider
runtime classes, 12 contract types, four registrations/manifests, CLI discovery,
configuration loading and startup through Composer's bin proxy. Lock SHA-256:
`1f8285856d486b205712dc8438610167c1ad74a532bbbaf3d1470a1634df2aa2`.

The provider PRs now carry formal comment reviews of this combined evidence.
The drafts remain open: these fixtures do not prove live permissions, database
authentication, timing, durable credential recovery, legacy migrations or lifecycle
concurrency. No PR was merged and no live provider operation was performed.

## Provider metadata example verification — 2026-10-16

All five provider `meta.json` files parse as JSON. Their exact config objects were
loaded through the CLI ConfigLoader and DTOs with synthetic environment values,
a fixture repository/domain and a local project path, then passed provider
validation. This verifies configuration shape and interpolation, not account
credentials, real server IDs, image availability or deployment success.

| Validation source | Revision |
| --- | --- |
| provider-forge | `f0c83e73b0d7078f970fa42d9e7b1fd13a8b7f1c` |
| provider-ploi | `5f5a41920eb5b773ec5a6163e5a4bf41ff23128b` |
| provider-coolify | `74957e46978731f4c05bb563c5754271309cf5f9` |
| provider-easypanel | `548f13a835b92c1f05b7ec7e0cb36c3b80a1bde9` |
| provider-cpanel combined | `225d7e9961b49bddfa1f7bbb9024517b675303f6` |

The Forge example previously omitted required `organization_slug`; it now also
shows the deployment timeout. The Ploi static-server example lacked `server_id`.
Coolify's three escaped dollar placeholders made its metadata invalid JSON.
Regressions reproduced all three defects before correction. EasyPanel's example
needed no change; a new no-API validation test passes (139 tests / 455 assertions
in its full suite). cPanel's source metadata matches the combined checkout byte
for byte; validation used that combined implementation, not an independently
released package. Its pre-existing untracked CLI fixture was not changed.

The website renders metadata fetched from provider main branches. These audit
branch examples do not appear there until merged and refreshed. Coolify is not
currently among the website's four configured provider repositories. No live API
calls or release/merge actions were performed for these example checks.

## Strategic observations

1. Current providers cluster around similar infrastructure assumptions.
2. The catalog is stronger for classic web hosting than for modern multi-service app platforms.
3. Shipper still risks sounding PHP-hosting oriented because the supported providers skew toward that ecosystem.
4. The product story becomes much stronger once SSH, Vercel, and one PaaS provider are added.

## Highest-priority gaps

1. Generic server deployments without a panel
2. Frontend and static hosting
3. PaaS application hosting
4. Multi-service application support
5. Clear rollback and preview-environment support model

## Recommended additions

| Provider | What gap it closes |
| --- | --- |
| SSH / Generic Server | True provider-agnostic fallback |
| Vercel | Frontend, static, edge workflows |
| Railway | PaaS apps, workers, managed services |
| Render | PaaS apps, web services, cron jobs |
| Complete and verify Coolify | Release the existing self-managed app-platform implementation |
| Kubernetes | Portable orchestration layer |

## Messaging implication

Website and docs should describe Shipper as:

- deployment-model agnostic
- provider-capability aware
- able to support different stacks depending on provider support

They should avoid implying:

- PHP-only deployments
- Git-required workflows
- panel-only infrastructure support


## Production consumer verification — 16 October 2026

An isolated Composer consumer installed six committed audit candidates from ZIP archives with `--no-dev --no-plugins --no-scripts`. Archive inspection excluded vendor, development tests, CI configuration and environment/authentication files. PHP ran with `-n` through the consumer wrapper. These are unmerged candidates, not released versions.

| Package | Verified commit |
| --- | --- |
| provider-forge | `0cf66004deb328aedf28e3da590f4ebf21c792ce` |
| provider-ploi | `4ea0d6f5d2bf33a6a098cc68b6b7cefa9cf9930a` |
| provider-easypanel | `54b19e235a1cda4bb6dbb9bd6d50720c73fffbfa` |
| provider-coolify | `36ffadd5e835a7354ac54167ebf004b4952d7370` |
| cli | `e16ba2b58b643e2de0272fe665d03f201db811ed` |
| contracts | `fe08db9199dd473d8de494dfa230fa53fc8bb446` |

Passed: 27 production runtime classes, 12 contract types, discovery/adaptation of all four installed providers, empty project configuration loading, CLI startup, and eight HTTP fixture suites (Forge, Ploi, EasyPanel, and five Coolify suites including status privacy). Fixtures were checked byte-for-byte against their committed source heads. Contracts used a `dev-audit as 1.2.0` compatibility alias for this consumer fixture.

Consumer lock SHA-256: `620f2e99c96d1f51505d8f6af29ee3acb462fdb24d7d209911d30ef9a946a8d6`. Local machine evidence: `/var/folders/hs/hh8zts3j2bx_m9649bfhwkc40000gn/T/shipper-production-verified-ydvfoofq` (`evidence.json`, `verification-results.json`, `install.log`). The temporary path is evidence location, not a portable reproduction command.

Separately, [cPanel PR #27](https://github.com/shippercli/provider-cpanel/pull/27) at `6cea43062bd86117cacf727a4afee95742e3fc2a` restricts saved Git tasks and retrieved deployment status records to summaries, removing commands and output. Its public-status regression failed before the change; 45 local tests / 216 assertions pass, and GitHub PHP 8.3–8.5 jobs pass. cPanel was not included in this six-package consumer.

Remaining limits: local fixtures do not prove live provider behavior, database connectivity, durable recovery after partial provisioning, concurrency safety, or queued deployment completion correlation for Ploi/EasyPanel. Other cPanel status sections retain existing behavior. No PRs were merged and no live resources were changed during this verification.


## Production consumer refresh after EasyPanel environment fixes

The 16 October audit was repeated with the following exact committed candidates. These remain unmerged, unreleased versions.

| Package | Commit |
| --- | --- |
| provider-forge | `0cf66004deb328aedf28e3da590f4ebf21c792ce` |
| provider-ploi | `4ea0d6f5d2bf33a6a098cc68b6b7cefa9cf9930a` |
| provider-easypanel | `7810a466f0a4cb97240886fbf90522e541400483` |
| provider-coolify | `36ffadd5e835a7354ac54167ebf004b4952d7370` |
| cli | `e16ba2b58b643e2de0272fe665d03f201db811ed` |
| contracts | `fe08db9199dd473d8de494dfa230fa53fc8bb446` |

A fresh ZIP-based Composer installation with no development dependencies, plugins or scripts passed runtime loading (27 classes), contract reflection (12 types), all four CLI provider adapters, empty-project configuration loading, and CLI startup under the PHP `-n` wrapper. All nine HTTP suites passed, including the six EasyPanel final-write/readback scenarios. Fixture PHP/Python files matched their committed provider heads. The same contracts compatibility alias described above was used.

Fresh Composer audit of the resolved consumer lockfile reports no advisories or abandoned packages. Lock SHA-256: `9de737f2af54fa15c81c0ce86d7de987aba3c9bf5a2a34c74e5b26501068a1db`. Local evidence directory: `/var/folders/hs/hh8zts3j2bx_m9649bfhwkc40000gn/T/shipper-production-verified-fyaamyso` (source/archive manifest, verification results, security audit and installation log).

EasyPanel now persists and verifies each confirmed database connection before creating another database, and verifies all expected variables after the final app update. Public-flow unit tests cover fresh-provider retry and refused readback; real HTTP covers final-write scenarios, with a negative control proving detection when verification is disabled. Worker/scheduler environment readback, durable crash recovery, ambiguous creation outcomes and live connectivity remain unresolved. cPanel was not installed in this consumer.

Additional independently checked candidates: website Markdown sanitization and reserved section-name navigation fixes; cPanel status/runtime summaries integrated locally with 755 tests / 3,371 assertions and four HTTP suites plus 27 archive tests; Python and Ruby dependency audit CI gates. These do not establish live account readiness or complete the overall audit.

### Production consumer refresh after EasyPanel worker and scheduler readback

The previous consumer check is superseded for EasyPanel by this fresh build. Six committed ZIP archives were installed with `--no-dev --no-plugins --no-scripts`; PHP ran through the isolated `-n` wrapper. Exact package heads:

| Package | Commit |
| --- | --- |
| provider-forge | `0cf66004deb328aedf28e3da590f4ebf21c792ce` |
| provider-ploi | `4ea0d6f5d2bf33a6a098cc68b6b7cefa9cf9930a` |
| provider-easypanel | `dbfc66438c39a5fe6ec5b12c48aca722809c79b5` |
| provider-coolify | `36ffadd5e835a7354ac54167ebf004b4952d7370` |
| cli | `e16ba2b58b643e2de0272fe665d03f201db811ed` |
| contracts | `fe08db9199dd473d8de494dfa230fa53fc8bb446` |

Runtime loading (27 classes), contract reflection (12 types), all four provider adapters, empty-project configuration loading and CLI startup passed. All ten HTTP fixture suites passed against the installed packages, including EasyPanel database/final-app and worker/scheduler environment readbacks. The worker/scheduler suite covers successful and refused/malformed/missing readbacks and confirms failed verification blocks the follow-on workload trigger. The bundled PHP 8.3–8.5 CI jobs passed on EasyPanel PR #14 at this head.

Composer audit of the resolved lockfile reported zero advisories and zero abandoned packages. Lock SHA-256: `be9aed598149c2781238a6edbe68ab3aa0bc16c51637394c194b98878d7bde44`. Local evidence directory: `/var/folders/hs/hh8zts3j2bx_m9649bfhwkc40000gn/T/shipper-production-verified-rrmckmeh` (source/archive manifest, verification results, audit and install log).

These fixtures do not prove live EasyPanel API behavior or workload health. Durable crash recovery, ambiguous creation outcomes and live connectivity remain unverified; cPanel is not included in this consumer. No PR was merged and no live resources were changed.
