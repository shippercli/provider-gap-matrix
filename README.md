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
| cPanel | Packagist [`1.2.1`](https://packagist.org/packages/shippercli/provider-cpanel#v1.2.1) | Validate, plan, apply, destroy, status, logs, rollback | Clean Composer and GitHub Action installs of v1.2.1 with CLI v1.0.4 deployed a raw HTML fixture and verified its content over trusted HTTPS. The v1.2.0 release commit passed the live interrupted-deploy retry check. |
| Ploi | Tagged GitHub repository `1.0.1` via Composer VCS | Validate, plan, apply, destroy, status, logs | Tagged package installation verified. Live provider operations still require a disposable Ploi target. |
| EasyPanel | Packagist [`2.0.0`](https://packagist.org/packages/shippercli/provider-easypanel#v2.0.0) | Validate, plan, apply, destroy, status, logs | Published v2.0.0 passed disposable deployments through [Composer](https://github.com/shippercli/provider-easypanel/actions/runs/43231219032/job/131226720750) and the [GitHub Action](https://github.com/shippercli/provider-easypanel/actions/runs/43231219032/job/131226720374), including trusted HTTPS and verified cleanup. |
| Forge | Source checkout | No stable package verified | Live provider operations still require a disposable Forge target. |
| Coolify | Source checkout | No stable package verified | Live provider operations still require a disposable Coolify target. |

The capability table below compares the original four providers at source level.
See the [Coolify coverage notes](https://github.com/shippercli/provider-coolify#coverage)
for its source-only capabilities.

## Coverage by deployment model

| Deployment model | Covered | Providers | Gap severity |
| --- | --- | --- | --- |
| Managed server panel | Yes | Ploi, Forge, EasyPanel | Low |
| Shared hosting panel | Yes | cPanel | Medium |
| Generic SSH / raw server | No | None | High |
| PaaS app platform | No | None | High |
| Frontend / static platform | No | None | High |
| Container-native platform | Partial | Coolify (source), EasyPanel (image deployments) | Medium |
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
| Daemon / process supervision | Yes | Yes | No | Yes | Medium |
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
