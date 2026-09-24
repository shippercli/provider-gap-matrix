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

## Coverage by deployment model

| Deployment model | Covered | Providers | Gap severity |
| --- | --- | --- | --- |
| Managed server panel | Yes | Ploi, Forge, EasyPanel | Low |
| Shared hosting panel | Yes | cPanel | Medium |
| Generic SSH / raw server | No | None | High |
| PaaS app platform | No | None | High |
| Frontend / static platform | No | None | High |
| Container-native platform | No | None | High |
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
| SSL certificates | Yes | Yes | Yes | Yes | Low |
| Database provisioning | Yes | Yes | Yes | Yes | Low |
| Environment variables | Yes | Yes | Yes | Yes | Low |
| Queue / worker management | Yes | Yes | No | No | Medium |
| Cron / scheduled jobs | Yes | Yes | Yes | No | Medium |
| Preview environments | Partial | No | Yes | Partial | Medium |
| Multi-service apps | No | No | No | Partial | High |
| Container image deploys | No | No | No | Partial | High |
| Rollbacks | Partial | No | Yes | No | High |
| Logs / runtime status | Partial | Yes | Yes | Partial | Medium |

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
| Coolify | Self-managed modern app platform |
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
