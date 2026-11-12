# Provider Gap Matrix

## Goal

Map current provider coverage, identify missing deployment models, and clarify where Shipper appears narrower than it should.

## Current providers

| Provider | Category | Primary model |
| --- | --- | --- |
| Ploi | Server panel | Managed VPS / server deployments |
| Laravel Forge | Server panel | Laravel-oriented server deployments |
| EasyPanel | App/server panel | Managed application and service deployments |
| cPanel | Shared hosting panel | Shared hosting deployments |
| Coolify | App/server panel | Self-managed container application deployments |

## Distribution and verification

| Provider | Stable installation | Stable CLI operations | Live deployment evidence |
| --- | --- | --- | --- |
| cPanel | Packagist [`1.2.3`](https://packagist.org/packages/shippercli/provider-cpanel#v1.2.3) | Validate, plan, apply, destroy, status, logs, rollback | Clean Composer and GitHub Action installs of v1.2.3 with CLI v1.0.5 validated, planned, deployed, and checked a live raw HTML fixture. The v1.2.0 release commit passed the live interrupted-deploy retry check. CLI v1.0.6 with provider v1.2.3 passed the live redeploy, backup, status, logs command, rollback, subdomain lifecycle, destroy, and account-resource teardown checks. Node.js and Python Passenger startup remain unverified. |
| Ploi | [`2.0.0`](https://github.com/shippercli/provider-ploi/releases/tag/v2.0.0) via Composer's Git driver | Validate, plan, apply, destroy, status, logs | CLI v1.0.6 installed the tagged provider through [Composer](https://github.com/shippercli/cli/pull/169) and validated its fixture through the [published GitHub Action](https://github.com/shippercli/actions/actions/runs/43291108683). Live operations require a disposable Ploi target. |
| EasyPanel | Packagist [`2.1.0`](https://packagist.org/packages/shippercli/provider-easypanel#v2.1.0) | Validate, plan, apply, destroy, status, logs | CLI v1.0.6 and provider v2.1.0 passed the published [Composer and GitHub Action Redis deployments](https://github.com/shippercli/provider-easypanel/actions/runs/43286913807), including authenticated Redis PING over trusted HTTPS and project removal. Both install paths also passed authenticated queries over trusted HTTPS, status, logs, and project removal for [MySQL](https://github.com/shippercli/provider-easypanel/actions/runs/43304680664), [MariaDB](https://github.com/shippercli/provider-easypanel/actions/runs/43305215848), [PostgreSQL](https://github.com/shippercli/provider-easypanel/actions/runs/43302680437), and [MongoDB](https://github.com/shippercli/provider-easypanel/actions/runs/43302151449). An [image-based preview run](https://github.com/shippercli/provider-easypanel/actions/runs/43288781847) verified orphan dry-run, exact removal, and preservation of a separate preview. Both published install paths also passed [public Git deployment over HTTPS and project removal](https://github.com/shippercli/provider-easypanel/actions/runs/43294122244). Both published install paths also passed [scheduled jobs](https://github.com/shippercli/provider-easypanel/actions/runs/43295458518) and [queue workers](https://github.com/shippercli/provider-easypanel/actions/runs/43295653031), including runtime checks and cleanup. The separate [connected GitHub-source fixture on main](https://github.com/shippercli/provider-easypanel/actions/runs/43295106806) returned HTTP 400 at deployment and verified project removal. |
| Forge | [`1.0.0`](https://github.com/shippercli/provider-forge/releases/tag/v1.0.0) via Composer's Git driver | Validate, plan, apply, destroy, status, logs | CLI v1.0.6 installed the tagged provider through [Composer](https://github.com/shippercli/cli/pull/169) and validated its fixture through the [published GitHub Action](https://github.com/shippercli/actions/actions/runs/43291108683). Live operations require a disposable Forge target. |
| Coolify | [`1.0.4`](https://github.com/shippercli/provider-coolify/releases/tag/v1.0.4) via Composer's Git driver | Validate, plan, apply, destroy, status, logs, rollback | On Coolify 4.7.5, CLI v1.0.6 with provider v1.0.4 installed through Composer passed nginx deployment and HTTP checks, Redis status and logs, repeat apply preserving the Redis UUID, and scoped teardown of both resources. With provider v1.0.3, the CLI also passed same-app redeploy and rollback. An explicit rollback restored a retained nginx tag. Automatic rollback restored a retained tag recorded in successful production history and refused to mutate the app when history lacked a usable tag. Standalone Dockerfile deployment and redeployment passed with [v1.0.2](https://github.com/shippercli/provider-coolify/releases/tag/v1.0.2). |

The capability table below compares the original four providers at source level.
See the [Coolify coverage notes](https://github.com/shippercli/provider-coolify/tree/v1.0.4#coverage)
for the tagged release's capabilities and limitations.

## Coverage by deployment model

| Deployment model | Covered | Providers | Gap severity |
| --- | --- | --- | --- |
| Managed server panel | Yes | Ploi, Forge, EasyPanel | Low |
| Shared hosting panel | Yes | cPanel | Medium |
| Generic SSH / raw server | No | None | High |
| PaaS app platform | No | None | High |
| Frontend / static platform | No | None | High |
| Container-native platform | Partial | Coolify, EasyPanel (image and public Git deployments) | Medium |
| Kubernetes | No | None | High |
| Preview-environment-first platform | No | None | Medium |

## Coverage by capability

Legend:

- `Yes`: first-class support should exist
- `Partial`: provider-dependent or uneven
- `No`: not covered today

| Capability | Ploi | Forge | cPanel | EasyPanel | Overall gap |
| --- | --- | --- | --- | --- | --- |
| App deployment | Yes | Partial | Yes | Yes | Medium |
| Domain management | Yes | Yes | Yes | Yes | Low |
| SSL certificates | Yes | Yes | Partial | Yes | Low |
| Database provisioning | Yes | Yes | Yes | Yes | Low |
| Environment variables | Yes | Yes | Yes | Yes | Low |
| Server lifecycle | Partial | No | No | No | High |
| PHP runtime configuration | Yes | No | Yes | No | Medium |
| Custom NGINX configuration | Yes | No | No | No | Medium |
| Queue / worker management | Yes | Yes | No | Yes | Low |
| Daemon / process supervision | Yes | Yes | No | No | Medium |
| Cron / scheduled jobs | Yes | Yes | Yes | Yes | Low |
| App resource limits | No | No | No | No | High |
| Service mounts | No | No | No | No | High |
| Preview environments | Partial | Partial | Partial | Partial | Medium |
| Multi-service apps | No | No | No | Partial | High |
| Container image deploys | No | No | No | Yes | High |
| Rollbacks | No | No | Partial | No | High |
| Logs / runtime status | Partial | Yes | Partial | Partial | Medium |

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
