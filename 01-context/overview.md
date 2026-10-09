# System context overview

> **Traceability:** This context overview is reconstructed from the supplied [data model](../spec/data-model.md), the challenge's only source. References to the model support factual statements. **[assumption]** identifies a likely interaction or context detail that the model does not define. This overview describes the system boundary; it is not an API, deployment, or user-interface specification.

## 1. System at a glance

Simple Stock Flow is an internal system for maintaining a product catalogue and available stock, recording completed sales, and calculating a sales report from those records (§1, §§2–3, §6.1). The model defines five entities: Category, Product, Sale, SaleItem, and User (§2). It has no Customer or Buyer entity, and a sale records the internal operator who made it (§1, §7).

The product catalogue is deliberately limited to product name, price, stock, category, and optional image (DP-03, §1). The five categories are seeded reference data and are read-only through the modeled application (§2.1, §9.1). A sale is an immutable business fact with one or more lines; line values are frozen at sale time so later catalogue changes do not rewrite them (§1, §§2.3–2.4, §7.1).

## 2. Context diagram

The diagram shows only actors and external stores supported by the data model. The interaction channel from operators to the application is **[assumption]** because the model does not define a UI, HTTP routes, or client protocol (§12).

```mermaid
flowchart LR
    admin[Admin]
    seller[Seller]
    deploy[Deployment manager<br/>[assumption: distinct actor]]
    app[Simple Stock Flow<br/>application boundary]
    db[(PostgreSQL 16<br/>simple_stock_flow<br/>schema sales)]
    images[(External image storage)]

    admin -->|Use internal capabilities<br/>[assumption: channel unspecified]| app
    seller -->|Record sales and use permitted capabilities<br/>[assumption: channel unspecified]| app
    deploy -->|Provide initial admin credentials<br/>from environment (§9.2)| app
    app -->|Read/write modeled entities;<br/>compute report (§2, §6.1)| db
    app -->|Store/resolve/delete binary<br/>by opaque key (D-08, §7.1)| images
```

There is no customer/buyer actor, payment system, message broker, separate report store, or asynchronous consumer in the modeled context (§1–3, §§6–7, §12). Their absence here is intentional and follows the supplied source, not an assertion that no future requirement could introduce one.

## 3. People and external parties

| Person or party | Relationship to the system | Context notes and evidence |
|---|---|---|
| **Admin** | Internal user with the `admin` role. | The initial admin is provisioned at startup from environment credentials; admins create sellers. Nobody grants the admin role during runtime user creation (§1, §9.2, §11 H-3 / DP-04). Exact permissions over catalogue and report operations are not fully stated; see **[assumption]** S-1 in the user-story backlog. |
| **Seller** | Internal user with the `seller` role who is associated with recorded sales. | The seller role and sale attribution are modeled (§1, §§2.3, 2.5, §7). A complete permission matrix is not provided. |
| **Deployment manager** | Operational party providing credentials for the initial administrator. | Environment-based startup provisioning is described in §9.2. Treating this as a distinct human role is **[assumption]**. |
| **External image storage** | Stores product-image binaries outside PostgreSQL. | Product records keep an opaque key, not image bytes or a path (D-08, §1, §7.1). The database and storage operations are not atomic (§7.1). |
| **Customer / buyer** | Not a participant in the modeled system. | No buyer identity or customer entity is included; sale attribution is to the internal operator (§1, §7). |

## 4. Information inside the boundary

### 4.1 Catalogue and reference information

The catalogue contains Product records with name, positive price, nonnegative stock, required category, and optional image key (§1, §2.2, DP-03). Product removal is a soft-delete state rather than physical deletion (§2.2, ADR-003). Category contains five fixed seeded records; the category repository is read-only (§2.1, §9.1, D-10).

### 4.2 Identity and access information

User records contain a unique username, a password hash, and one of two roles: `admin` or `seller` (§1, §2.5). The domain does not receive a clear-text password; the hash is produced through a hash port (D-09, §2.5). Usernames and sale attribution are classified as personal data, roles as internal-confidential, and password hashes as authentication secrets (§7).

### 4.3 Sales and derived information

Sale records the operator, time, and line items (§1, §2.3). SaleItem belongs only to its Sale and stores a quantity, frozen product name, frozen unit price, and a frozen category label as described in §2.4. The physical placement/status of `category_name` is inconsistent elsewhere in the model; the architecture consistency check records the adopted interpretation and remaining uncertainty (CC-02/CC-03).

Sale totals and line subtotals are calculated rather than stored as columns. The sales report is a computed read model for a date range, not a separate persisted entity (§1, D-06, §6.1). The report does not break down by seller (DP-02, §7.1). Its frozen-category grouping can return multiple rows for one product after recategorization; this conflicts with the source criterion called out in CC-10 / §11.1.

## 5. System responsibilities and boundary behavior

The system is responsible for preserving the modeled domain rules across the boundary between application logic and PostgreSQL:

- Keep product stock nonnegative and reject withdrawals above available stock (§2.2, §4, ADR-002).
- Coordinate adding a sale line with withdrawing stock, require at least one line before confirmation, and preserve sales as immutable records (§2.3, §7.1).
- Store historical sale values as snapshots rather than reading later values from the current catalogue (§1, §2.4).
- Keep the database image reference consistent with the external binary lifecycle: clear and commit the key before deleting the binary (§7.1).
- Avoid exposing authentication secrets or expanding data collection beyond the modeled identity and sale-attribution purpose (§7, §12).

These responsibilities do not mean every rule is currently enforced by PostgreSQL. The model labels each rule as engine-enforced, domain-only, or pending; positive price, positive quantity, valid role, and username normalization are domain-only with T-20 recorded for engine checks (§2.1–2.5, §4). Sale attribution's User foreign key is pending T-12 (§3, §5). Product search indexes are associated with T-13 (§6.2). See the architecture consistency check for stale or conflicting schema statements (CC-01–CC-07).

## 6. Context constraints

The following constraints shape the system context and limit what can be promised:

1. **Single currency:** amounts use the system's one currency; no currency column is modeled (D-05, §3, §12).
2. **Fixed product shape:** product attributes are limited to name, price, stock, category, and optional image (DP-03, §1, §12).
3. **Internal identity only:** there is no customer/buyer identity; a sale records an operator (§1, §7).
4. **Historical retention:** sales are immutable and retained indefinitely; products use soft deletion (§2.2–2.3, §7.1).
5. **No catalogue audit timestamps:** `created_at` / `updated_at` columns are explicitly excluded (§8).
6. **No defined messaging boundary:** no event broker, outbox, or asynchronous communication is specified (§2, §6, §12).
7. **No operational targets:** availability, recovery, workload capacity, and numeric performance objectives are not established by the model (§12; NFR-13).

## 7. Assumptions and open context questions

| ID | Assumption or question | Why it is open | Evidence / follow-up |
|---|---|---|---|
| CTX-A1 | **[assumption]** Admins and sellers use an application interface to access system capabilities. | The model gives domain and query capabilities, but does not describe screens, routes, or protocol. | §6.1, §12. |
| CTX-A2 | **[assumption]** The deployment manager is a distinct operational actor. | Startup credentials are specified, but the person/team responsible is not. | §9.2. |
| CTX-A3 | **[assumption]** Admins manage the catalogue and reports, while sellers register sales. | Roles are defined but their full operation permissions are not. | User-story backlog S-1; confirm with owner. |
| CTX-Q1 | Should the source report criterion be rewritten to allow one row per product and frozen category label? | The owner decision and source criterion conflict. | §11.1, CC-10. |
| CTX-Q2 | Who owns orphan-image cleanup? | The image deletion sequence may leave an orphan binary; no cleanup process is defined. | §7.1, §11 H-2. |
| CTX-Q3 | What are the required availability, recovery, capacity, and response-time targets? | The model specifies none. | §12, NFR-13. |

## 8. Related context documents

- [Scope](scope.md) details included capabilities, exclusions, constraints, and pending questions.
- [Context glossary](glossary.md) defines the canonical terms used in this overview.
- [Domain entities and rules](../02-domain/entities-and-rules.md) describes aggregate boundaries and invariant enforcement.
- [Product problem framing](../03-product/problem-framing.md) explains the problem implied by the integrity and history rules.
- [Product vision](../03-product/vision.md) states the intended product direction as an explicitly marked inference.
