# Product vision

> **Traceability:** This vision is reconstructed from the supplied [data model](../spec/data-model.md), the only source for the challenge. Model references are cited throughout. **[assumption]** marks product framing or a proposed success measure that the model does not state as a requirement. This document does not claim to reproduce an unavailable product brief.

## 1. Vision statement

**[assumption]** Simple Stock Flow helps internal operators maintain a dependable product catalogue, register completed sales with the corresponding stock movement, and review sales history whose recorded values remain faithful to what was sold.

The model supports that vision through a Product aggregate that owns catalogue and stock rules, a Sale aggregate that owns completed sales and their lines, and a computed report read model (§§1–2, §6.1). Product and sale-line values are related by snapshots: later changes to the catalogue do not update the historical name and price recorded on a sale (§1, §2.4). The report is calculated from sales rather than persisted as a separate business entity (D-06, §1).

## 2. Product purpose

The product brings three connected responsibilities into one model:

1. **Maintain a small, controlled catalogue.** Products have a deliberately bounded set of attributes: name, price, stock, category, and optional image. Categories come from five seeded reference values and are read-only (§1, §§2.1–2.2, §9.1, DP-03).
2. **Record sales as durable business facts.** A sale records its operator, time, and one or more sale lines. Adding a line coordinates stock withdrawal; a sale cannot be confirmed without a line, and registered sales are not edited or deleted (§1, §2.3, §7.1).
3. **Support historical review.** Sale lines preserve values from the time of sale. A report can aggregate sales for a date range without changing old sale records when the catalogue changes (§1, §2.4, D-06, §6.1). For category changes, the owner's recorded decision is to group by the frozen category label, which can yield more than one report row for a product; the related source-spec conflict remains declared in CC-10 / §11.1.

**[assumption]** The product is intended to make routine catalogue and sales work more reliable for an internal operation. The data model explains integrity and history controls, but does not quantify current operational errors, the size of the business, or financial impact.

## 3. Users and the value they need

| User or party | Need addressed by the product | Evidence and boundary |
|---|---|---|
| **Administrator** | Maintain catalogue data, manage seller accounts, and consult sales information. | The `admin` role exists, and an administrator creates sellers (§1, §2.5, §11 H-3 / DP-04). The exact permission matrix for catalogue and report operations is not defined; those assignments in the user-story backlog are **[assumption]** S-1. |
| **Seller** | Record sales under their own identity while stock is withdrawn and the sale is preserved. | The `seller` role exists and a sale records who made it (§1, §2.3, §2.5). Exact access to catalogue operations is not specified. |
| **Deployment manager** | Provision the first administrator using environment-supplied credentials. | Startup provisioning and environment credentials are described in §9.2. Treating this as a distinct actor is **[assumption]**. |
| **Customer or buyer** | No customer-facing need is addressed by this model. | The system records an internal operator, not a buyer; no customer entity or buyer data is modeled (§1, §7). |

The model's identity boundary is intentionally internal: usernames and seller attribution are personal data, while password hashes are authentication secrets with stricter handling rules (§7). The product does not define a public customer account or customer purchase history (§1, §7, §12).

## 4. Product outcomes

The following outcomes describe what the model makes possible. They are not numeric service-level objectives.

| Outcome | Product behavior implied by the model | Evidence |
|---|---|---|
| **Stock remains valid.** | Persisted stock cannot be negative, and a withdrawal beyond available stock is rejected by domain behavior. | §2.2, §4, ADR-002. The database check is the last barrier for negative values; over-withdrawal itself is a domain rule. |
| **A sale and stock movement agree.** | Adding a sale line and withdrawing its product's stock are one business operation. | §2.3. **[assumption]** A database transaction should implement the operation atomically; the transaction policy is not explicitly stated. |
| **Past sales remain trustworthy.** | Product name and unit price are frozen on sale lines; sales have no edit/delete operation. | §1, §§2.3–2.4, §7.1. The report also uses the frozen category label according to §11.1, subject to CC-10. |
| **Product removal preserves references.** | Products leave active catalogue reads through soft deletion instead of physical deletion. | §2.2, ADR-003, §7.1. |
| **Reports are derived from recorded sales.** | A date-range report is computed through a read port rather than stored in a report table; it is not broken down by seller. | §1, Q9 / §6.1, D-06, DP-02. |
| **Sensitive credentials stay protected.** | The domain works with a password hash and does not receive the clear-text password; the hash is never exposed in logs or responses. | D-09, §2.5, §7. |

## 5. Product principles

### 5.1 Protect the integrity of the sale

A sale is a business fact composed of its sale lines, and line creation coordinates stock withdrawal (§2.3–2.4). Maintain the aggregate rule and the database constraints that protect stock and relationships (§2.2, §4–5). **[assumption]** When concurrent changes conflict, surface a conflict for a fresh read rather than silently overwriting another update; the model establishes optimistic concurrency with `xmin` but does not prescribe the user-facing conflict flow (D-04, §3).

### 5.2 Preserve what happened

Treat a completed sale as immutable and retain it with its lines indefinitely (§1, §§2.3–2.4, §7.1). Store snapshots of values that may change in the catalogue so the historical record does not follow the current product (§1, §2.4). Apply the owner's frozen-category grouping decision in report behavior, while keeping the contradiction with the signed source criterion visible (§11.1, CC-10).

### 5.3 Minimize the information collected

Collect the operator identity needed for authentication and sales attribution, but do not add buyer data or a seller-level report breakdown (§1, §7, DP-02). Do not expand Product beyond its decided fields, add currency columns, or add `created_at` / `updated_at` without a new decision (DP-03, D-05, §§8, 12).

### 5.4 Keep simple reference data simple

Use the five fixed categories seeded by the initial migration. The category repository is read-only; category maintenance is not part of the product's current scope (§2.1, §9.1, D-10). **[assumption]** If category administration becomes a real need, revisit uniqueness behavior and its case/accent sensitivity before adding category CRUD (§4.1).

### 5.5 Keep storage responsibilities explicit

The database stores an opaque image key, while the binary lives in external storage (D-08, §1, §7.1). Clear and commit the key before deleting the binary; database and image-storage operations are not atomic together (§7.1). Orphan-binary cleanup has no defined owner or process (§11 H-2).

### 5.6 Treat quality gaps honestly

Some rules are domain-only today, some changes are tracked as pending tasks, and some outputs in the model were measured on an earlier date (§2, §§4–6, §§10, 13). Do not present pending checks, indexes, foreign keys, or operational targets as already delivered; use the consistency findings CC-01–CC-10 to qualify disputed state.

## 6. Product capabilities

The vision is supported by the following capability groups, not by a particular screen layout or API design:

- **Identity and access:** internal users have one of two roles; the first admin is provisioned at startup, and admins create sellers (§1, §2.5, §9.2, §11 H-3 / DP-04). **[assumption]** The interface uses an authentication mechanism that can enforce the observed unauthenticated and forbidden outcomes; token format is not specified (§13 D-3).
- **Catalogue and stock:** read the fixed categories; create, find, update, restock, attach an image to, and soft-remove products based on the modeled aggregate and access patterns (§2.1–2.2, Q1–Q5 / §6.1, D-08, ADR-003). Exact role permissions remain **[assumption]** S-1.
- **Sales:** register and read sales with lines, support date-range lookup, and preserve the product values copied at sale time (§2.3–2.4, Q6–Q7 / §6.1).
- **Reporting:** calculate sales aggregation for a date range using the report read port and the frozen data; do not persist a separate report or break it down by seller (D-06, Q9 / §6.1, DP-02, §11.1).

The model does not define routes, request/response formats, screens, localization, or an exact error contract (§12). Any such design is **[assumption]** until specified elsewhere.

## 7. How success could be recognized

The data model provides invariants and examples of measured database state, but it does not set product KPIs or acceptance thresholds (§§2–4, §10). The following are therefore **[assumption]** candidate outcome checks, not commitments:

1. A completed sale has at least one line, and the stock change agrees with the sold quantities (§2.2–2.3).
2. No persisted product has negative stock, including after concurrent updates or writes outside the normal application path (§2.2, D-04, ADR-002).
3. Editing a product after a sale does not alter the name or unit price already recorded on its sale line (§1, §2.4).
4. A report uses the defined date range and frozen labels, does not expose a seller breakdown, and is computed from sales data (§1, §6.1, DP-02, §11.1).
5. Password hashes do not appear in responses, logs, projections, or errors (§7).

No numerical thresholds for latency, availability, recovery, capacity, or usability are proposed because the model provides none (§12; see NFR-13 in [`../04-requirements/non-functional.md`](../04-requirements/non-functional.md)).

## 8. Explicit non-goals

This vision does not promise:

- Customer or buyer accounts, customer identity, or customer-related sale history (§1, §7).
- Multi-currency sales, a currency column, or multiple price currencies (D-05, §3, §12).
- A report grouped or filtered by seller (DP-02, §6.3, §7.1).
- Additional product fields such as description, SKU, or reference code (DP-03, §1, §12).
- Category creation, rename, or deletion through the application (§2.1, §9.1).
- Editing or deleting recorded sales, or physically deleting products (§2.2–2.3, §7.1).
- A catalogue audit trail through `created_at` / `updated_at` columns (§8).
- A durable domain-event stream, message broker, or event-sourced history; none is defined by the model (§2, §6, §12).
- Guaranteed orphan-image cleanup, since its policy is unresolved (§11 H-2).

## 9. Assumptions and open decisions

| ID | Assumption or open point | Why it remains open | Source |
|---|---|---|---|
| PV-A1 | **[assumption]** The product is used by an internal operation with administrators and sellers. | The roles and sales attribution are modeled, but organization context is not described. | §1, §§2.3, 2.5 |
| PV-A2 | **[assumption]** The user interface supports catalogue work, sale registration, and report review. | Capabilities and query patterns exist; no UI is specified. | §6.1, §12 |
| PV-A3 | **[assumption]** Sale persistence should be wrapped in one database transaction. | Sale and stock movement are one business operation, but the transaction boundary is not written as a persistence guarantee. | §2.3 |
| PV-A4 | **[assumption]** Administrators manage catalogue changes and reports while sellers register sales. | The model defines roles but not a complete permission matrix; the backlog records this as S-1. | §1, §2.5; user-stories.md S-1 |
| PV-O1 | Rewrite the source report criterion from “one row per product” to include the frozen label. | The model says the statements conflict and leaves the signed source-spec change to the owner. | §11.1, CC-10 |
| PV-O2 | Define availability, backup/recovery, capacity, and latency objectives if required. | The data model sets no such targets. | §12; NFR-13 |
| PV-O3 | Decide who owns cleanup of orphan image binaries. | The deletion sequence can leave an orphan and no cleanup process is defined. | §7.1, §11 H-2 |

## 10. Traceability summary

| Vision element | Model evidence |
|---|---|
| Catalogue scope and fixed categories | DP-03, §§1, 2.1–2.2, §9.1 |
| Stock correctness and sale operation | §§2.2–2.3, §4, ADR-002, D-04 |
| Immutable sale history and frozen values | §§1, 2.3–2.4, §7.1 |
| Internal roles and credential handling | §§1, 2.5, 7, 9.2, §11 H-3 / DP-04, §13 D-3 |
| External images and retention order | D-08, §7.1, §11 H-2 |
| Computed report and frozen-label decision | D-06, Q9 / §6.1, DP-02, §11.1 / CC-10 |
| Explicit exclusions and scope limits | DP-02/03, D-05, §§1, 8, 12 |
