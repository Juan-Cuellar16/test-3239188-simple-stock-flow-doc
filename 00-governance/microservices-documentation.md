# Microservices Documentation Standard

> **Current applicability:** Simple Stock Flow is documented as a single-service system. This standard records that boundary and defines what must be decided if the architecture changes. It does not require creating per-microservice documents for the current challenge.
>
> **Traceability:** Claims are based on the supplied [data model](../spec/data-model.md) and [architecture overview](../05-architecture/overview.md). Future service boundaries and documentation requirements marked **[assumption]** need an approved architecture decision.

## 1. Purpose

This document prevents the per-microservice structure from being applied when the modeled system does not define independently deployed microservices. It also provides a starting checklist in case an approved future design introduces them.

The challenge currently asks for repository-level documentation in `01-context` through `05-architecture`; `spec/data-model.md` is the supplied data source. This standard does not expand those deliverables.

## 2. Current architecture boundary

The current architecture overview describes one application service, one PostgreSQL database, and external image storage. The model provides read/write access patterns for the system and a separate report read port; it does not define separate service-owned databases, a service catalog, a message broker, or asynchronous service communication (§6.1, §12; `05-architecture/overview.md` §§3–4, 6).

The image-storage dependency holds binaries addressed by opaque keys; it is an external storage boundary, not evidence of a separate business microservice (D-08, §1, §7.1).

### Current applicability

| Documentation item | Status for the current system | Reason |
|---|---|---|
| Per-microservice README | Not required | No independently deployed business microservices are defined (§12). |
| Per-service data model | Not required | The supplied `spec/data-model.md` is the system's data-model source. |
| Per-service event catalog | Not required | No event publication or asynchronous messaging is specified (§6, §12). |
| Per-service decisions file | Not required | No per-service boundary or service-specific decision record is defined. |
| Per-service runbook | Not required by this challenge | Operational deployment and runbook requirements are not specified (§12). |
| OpenAPI contract per service | Not established | The model does not define HTTP routes or API contracts (§12). |

## 3. Current documentation locations

Use the challenge's existing repository-level structure:

- `01-context/` — system context, scope, and glossary.
- `02-domain/` — entities, rules, and candidate domain events.
- `03-product/` — problem framing and product vision.
- `04-requirements/` — stories, non-functional requirements, and traceability.
- `05-architecture/` — architecture overview and consistency check.
- `spec/data-model.md` — supplied data-model source; do not replace it.

The challenge README defines which files are required. Governance files in `00-governance/` are supplemental and do not change the submission requirements.

## 4. Do not infer microservice architecture

Do not create service folders, assign database ownership to hypothetical services, or describe inter-service contracts as current system facts without an approved architecture decision.

In particular:

- A repository, adapter, port, or database table is not by itself a microservice boundary.
- The report read port does not establish a separate reporting service; the model describes a computed read model (D-06, §1, §6.1).
- Candidate domain-event names do not establish a broker, outbox, or integration-event contract; see `02-domain/domain-events.md` and model §12.
- External image storage does not imply a separate business service (D-08, §7.1).

## 5. Future per-service documentation checklist

If an approved architecture later creates independently deployed services, **[assumption]** each service should have a documentation set proportionate to its responsibilities:

| Document | Minimum content to define |
|---|---|
| `README.md` | Service purpose, owners, responsibilities, interfaces, dependencies, and local operation. |
| `data-model.md` | Data owned by the service, constraints, migrations, and retention rules. |
| `decisions.md` | Service-specific decisions and links to system-level ADRs. |
| `events.md` | Events produced or consumed, schemas, ownership, ordering, delivery, and failure behavior. Required only if the service actually uses events. |
| `runbook.md` | Deployment, health checks, recovery, common failures, and operational contacts. Required when operational support is defined. |
| API contract | Routes, request/response schemas, authentication, and errors, if the service exposes an API. |

These are proposed future documentation expectations, not current Simple Stock Flow requirements.

## 6. Conditions for revisiting this standard

Revisit this document only when an approved architecture decision proposes a service boundary. Before calling a component a microservice, document:

1. Its business responsibility and bounded context.
2. Its data ownership and the rule against shared database ownership.
3. Its independent deployment and operational ownership.
4. Its synchronous and asynchronous contracts, if any.
5. Its security boundary, failure handling, and observability needs.
6. The impact on current aggregates, repositories, and report access patterns.
7. The required per-service documentation and its owner.

**[Assumption]** A service split should not be approved solely to create folders or satisfy this template; it should address a documented business or operational need.

## 7. Open decisions

| Decision | Status |
|---|---|
| Are independently deployed business microservices part of the current system? | No such boundaries are defined by the supplied model or current architecture overview. |
| Who approves a future service split? | **[TODO: identify the project owner or review role]** |
| What deployment, operational, and API documentation is required if a split occurs? | **[TODO: decide with the future architecture]** |
| Does any future service require asynchronous event delivery? | **[TODO: define a consumer and delivery need before adopting a broker or outbox]** |