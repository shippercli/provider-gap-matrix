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

A fresh GitHub query inspected all 48 open PRs, including their current head,
draft state, mergeability, review decision and every reported status check.
All 136 reported checks were successful; none were pending or failing. GitHub
reported all 48 heads mergeable without conflicts. There were 18 ready PRs and
30 drafts. This is a dated observation, not a guarantee for later commits.

Twenty PRs still reported a blocked merge state: cPanel #8–#25, contracts #5
and capability-spec #2. In cPanel, branch protection requires `CI / PHP 8.3`,
`CI / PHP 8.4` and `CI / PHP 8.5`, while the older PR workflows report
`PHP 8.3`, `PHP 8.4` and `PHP 8.5`. The existing one-line
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
