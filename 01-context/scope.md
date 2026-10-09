# Scope

> **Traceability:** This scope is reconstructed from the supplied [data model](../spec/data-model.md), the only source for the challenge. Model sections and decision identifiers are cited for each boundary. **[assumption]** marks inferred user-facing behavior or an operational recommendation that the model does not establish.

## 1. Scope statement

Simple Stock Flow covers an internal product catalogue, available stock, internal user identity, completed sales, and a sales report calculated from recorded sales (§§1–2, §6.1). Its data model has five entities—Category, Product, Sale, SaleItem, and User—and no separate report or customer entity (§1–3).

The scope is intentionally narrow. Products have a fixed set of attributes, categories are a seeded read-only set, sales are immutable, and the report is not broken down by seller (DP-02/03, §§1, 2.1–2.4, §9.1). Features beyond those boundaries need a separate requirement and decision; they should not be inferred from the presence of a database or an architectural port.

## 2. In-scope capabilities

| Capability | What is included | Model evidence and limits |
|---|---|---|
| **Internal identity** | Users authenticate as one of two roles: `admin` or `seller`. The first admin is provisioned at application startup using environment credentials; admins create sellers. | §1, §2.5, §9.2, §11 H-3 / DP-04. Exact role permissions for catalogue and reporting are not fully specified; the backlog records those assignments as **[assumption]** S-1. |
| **Category reference data** | Read the five seeded categories and associate products with an existing category. | §2.1, §2.2, §9.1, D-10. No application operation creates, renames, or deletes categories. |
| **Product catalogue** | Maintain products with name, positive price, nonnegative stock, required category, and optional image reference. Search and read products using the model's access patterns. | §1, §2.2, Q1–Q5 / §6.1, DP-03. Some rules are domain-only or have engine changes tracked in T-20 (§4). |
| **Stock operations** | Restock products and withdraw units when a sale is registered, while preserving the nonnegative-stock invariant. | §2.2–2.3, §4, ADR-002. The model does not define inventory adjustments beyond the Product behaviors it names. |
| **Product removal** | Remove products from active catalogue reads through soft deletion; retain the underlying row and historical references. | §2.2, ADR-003, §7.1. The consistency check records stale T-09 prose (CC-05). |
| **Product images** | Store image binaries externally; keep only an opaque image key in the product. Clear and commit the key before deleting the binary. | D-08, §1, §7.1. Database and binary deletion are not atomic; orphan-binary cleanup is unresolved (§11 H-2). |
| **Sales** | Register an immutable sale with at least one line; associate the sale with an internal operator and time; preserve product values copied at the sale moment. | §1, §§2.3–2.4, §3, §7.1. The Sale aggregate coordinates line creation and stock withdrawal (§2.3). |
| **Sales reads** | Read a sale with its lines and list sales for a date range. | Q6–Q7 / §6.1. End-date inclusivity is not settled in the model or current story backlog. |
| **Sales report** | Calculate a sales aggregation for a date range through a read port; do not persist a report entity. Do not group or filter by seller. | §1, Q9 / §6.1, D-06, DP-02. The category grouping follows the frozen-label decision in §11.1, which conflicts with the source criterion recorded in CC-10. |

**[assumption]** These capabilities are available through an application interface to internal operators. The data model does not define screens, routes, request/response formats, or an authentication protocol (§12).

## 3. People and system boundary

| Party or system | Scope relationship | Evidence and qualification |
|---|---|---|
| **Admin** | Internal user role; creates sellers. | §1, §2.5, §11 H-3 / DP-04. The initial admin is provisioned from deployment environment credentials (§9.2). |
| **Seller** | Internal user role; identity is associated with a sale. | §1, §§2.3, 2.5, §7. The full permission set is not defined. |
| **Deployment manager** | Supplies the first admin's environment credentials. | §9.2. Treating this as a distinct human actor is **[assumption]**. |
| **External image storage** | Stores image binaries referenced by opaque keys. | D-08, §1, §7.1. It is outside the database transaction boundary. |
| **Customer / buyer** | Outside the modeled system. | No buyer entity or buyer data is present; the recorded person is the internal operator (§1, §7). |
| **Payment service / payment data** | Not part of the defined scope. | The model defines no payment entity or payment integration (§1–3, §12). |

## 4. Data and lifecycle boundaries

- **Catalogue:** the Product attribute set is closed to name, price, stock, category, and optional image (DP-03, §1). Product removal is logical, not a physical row deletion (§2.2, ADR-003).
- **Categories:** the five reference rows are seeded in the initial migration; they are not maintained through the product interface (§2.1, §9.1, D-10).
- **Sales:** a Sale contains one or more SaleItems, and sale values remain frozen after catalog changes. Sales and their lines are retained indefinitely (§1, §§2.3–2.4, §7.1).
- **Identity:** the system stores internal username, role, and password hash. It does not store a clear-text password or customer identity (§2.5, §7, D-09).
- **Images:** database state stores only an opaque key; the binary is stored outside the database. The deletion order can leave an orphan binary if the external delete fails (§7.1, §11 H-2).
- **Reports:** the report is a computed read model, not a stored business entity (§1, D-06, §6.1). It does not expose seller-level breakdowns (DP-02, §7.1).

## 5. Explicitly out of scope

The model excludes or does not define the following capabilities:

| Excluded capability | Boundary | Evidence |
|---|---|---|
| Customer/buyer accounts and sale attribution to a buyer | Sales record an internal operator; no customer entity or buyer data is modeled. | §1, §7 |
| Multiple currencies | The system is monocurrency and has no currency column. | D-05, §3, §12 |
| Seller performance or seller-grouped reports | The report is not broken down by operator. | DP-02, §6.3, §7.1 |
| Additional product attributes such as SKU or description | Product attributes are explicitly limited to the decided set. | DP-03, §1, §12 |
| Category create/rename/delete | Categories are fixed seeded reference data with a read-only repository. | §2.1, §9.1, D-10 |
| Editing or deleting a registered sale | Sales are immutable and retained indefinitely. | §1, §2.3, §7.1 |
| Physical product deletion | Product removal is soft deletion; a sale line must not be orphaned by a physical delete. | §2.2, §5, ADR-003 |
| Catalogue audit timestamps | `created_at` / `updated_at` columns are explicitly not part of the model. | §8 |
| Durable event publication, event sourcing, or a message broker | No event store, outbox, broker, or asynchronous consumer is defined. | §2, §6, §12 |
| Availability, backup/recovery, capacity, and numeric performance commitments | No SLO, RPO/RTO, workload size, or latency threshold is defined. | §12; NFR-13 |

The absence of these features in the model is a scope boundary for this challenge, not a claim that the business could never request them.

## 6. Constraints and known implementation gaps

Scope statements must distinguish business capabilities from the current strength of their implementation:

1. **Domain-only rules:** positive product price, positive sale quantity, non-empty category name, valid role, and lowercase username are currently classified as domain-only; T-20 is associated with moving these rules into the database (§2.1–2.5, §4).
2. **Sale attribution:** `sold_by` is currently text. The relational user reference FK-4 and associated `sold_by_user_id` work are pending T-12 (§3, §5).
3. **Indexes:** product search/report indexes are tracked in T-13. The model has conflicting/stale index inventories; use CC-04/CC-06 when describing exact current state (§6.2, §10.3).
4. **Frozen category label:** the model's §2.4 says `sale_item.category_name`, while §3 has a contradictory Product-column entry. CC-02 adopts the reading that the label belongs to `sale_item`; CC-03 leaves its exact engine status uncertain.
5. **Report grouping:** the owner's decision groups by frozen category label, potentially returning multiple rows per product after recategorization. The original “one row per product” criterion conflicts and still requires a source-spec rewrite (§11.1, CC-10).
6. **Operational definition:** authentication transport, UI, error contract, availability, recovery, and numeric service targets are not established by the model (§12; NFR-13).

These known gaps do not automatically remove the associated user capability from scope. They describe constraints and work that must remain visible rather than being presented as completed guarantees.

## 7. Assumptions and owner questions

| ID | Assumption or question | Reason | Status / source |
|---|---|---|---|
| SC-A1 | **[assumption]** Internal operators use an application interface to access catalogue, sales, and report capabilities. | No interface is defined in the model. | Confirm if a channel must be named; §12. |
| SC-A2 | **[assumption]** Admins maintain the catalogue and read reports; sellers register sales. | Roles are defined, but a complete permission matrix is not. | Backlog S-1; owner confirmation pending. |
| SC-A3 | **[assumption]** Sale data and stock changes should be committed as one transaction. | The aggregate defines one operation but persistence transaction policy is not explicit. | §2.3; NFR-02. |
| SC-Q1 | Should the report source criterion be rewritten as “one row per product and frozen label”? | Owner decision and signed source criterion currently conflict. | §11.1, CC-10. |
| SC-Q2 | Who owns orphan-image cleanup? | No cleanup process is defined. | §7.1, §11 H-2. |
| SC-Q3 | What availability, recovery, capacity, and latency targets are needed? | The data model supplies no operational objectives. | §12, NFR-13. |
| SC-Q4 | Is the sales date-range end inclusive? | The model names a range but does not specify its boundary semantics. | §1; HU-015 open question. |

## 8. Traceability summary

| Scope area | Model evidence |
|---|---|
| Entities and allowed Product attributes | §1, §2.1–2.5, §3, DP-03 |
| User roles and first-admin provisioning | §1, §2.5, §9.2, §11 H-3 / DP-04 |
| Stock integrity and sale lifecycle | §2.2–2.4, §4, §7.1, ADR-002 |
| Category seed data and image storage | §2.1, §9.1, D-08, §7.1 |
| Read/query/report capabilities | Q1–Q10 / §6.1, D-06, DP-02, §11.1 |
| Excluded fields, currencies, and audit columns | D-05, DP-02/03, §§1, 8, 12 |
| Known uncertainties | §10, §11, §13 and CC-01–CC-10 in the architecture consistency check |
