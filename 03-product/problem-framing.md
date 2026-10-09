# Problem framing

> **Traceability:** This framing is reconstructed from the supplied [data model](../spec/data-model.md), the challenge's only source. Each factual claim cites its model evidence. **[assumption]** marks an inference about users, impact, or business context that the model does not state. The model describes controls and gaps; it does not include incident history or a current-state process study.

## 1. Problem statement

**[assumption]** An internal stock-and-sales operation needs a reliable way to maintain product information and available quantities while recording completed sales. If catalogue changes, stock movement, and sale history are not handled consistently, operators may see stock that is not actually available or historical sales that no longer reflect what was sold.

The model identifies concrete rules intended to prevent these data-integrity problems: stock must remain nonnegative, a sale line withdraws stock, and sale-line values are copied from the product at the time of sale (§§1, 2.2–2.4, 4). Sales are immutable and retained, while products are removed through soft deletion (§2.2–2.3, §7.1, ADR-003). These controls establish the problem this product must address; they do **not** prove that the described failures have occurred in production.

## 2. Context and evidence

The modeled system contains five entities: Category, Product, Sale, SaleItem, and User (§2). Products have a bounded set of fields—name, price, stock, category, and optional image—and categories are a fixed set of five seeded values (§1, §2.1–2.2, §9.1, DP-03). A sale records an internal operator, time, and one or more lines; there is no customer or buyer entity (§1, §2.3, §7).

The data model documents several integrity mechanisms and known limitations:

- Negative stock is prevented by a database check, while rejecting a withdrawal greater than available stock is a domain rule (§2.2, §4, ADR-002).
- Sale creation coordinates stock withdrawal with adding a sale line; a sale needs at least one line before it can be confirmed (§2.3).
- Product name and price are frozen on the sale line, and a category label is also specified as frozen (§1, §2.4). The physical-column placement and current state of `category_name` conflict in the model; see CC-02 and CC-03 in the consistency check.
- Optimistic concurrency uses PostgreSQL `xmin` (D-04, §3). The model does not define a user-facing conflict or retry policy.
- Some invariants are enforced only in the domain, and several engine checks or indexes have planned tasks (§§2, 4, 6). The model itself notes stale measurements and contradictions; see CC-01–CC-10.

This is evidence of modeled risks and safeguards, not a report of actual incidents. The supplied material gives no transaction volume, error rate, current workflow, user interviews, financial loss, or operational baseline.

## 3. Problem dimensions

### 3.1 Stock can become inconsistent with sales

A product's stock is an available-unit count and must never be negative (§1, §2.2). A sale line reduces that stock, and the model says the withdrawal and line addition form one operation (§2.3). The database check catches negative values, but the rule that a sale cannot withdraw more than is available lives in domain behavior (§2.2, §4).

**Problem to solve:** preserve the relationship between a recorded sale and the stock movement it causes, including when persistence fails or concurrent operations compete for the same stock (§2.3, D-04, ADR-002).

**[assumption]** If this relationship is broken, operators could rely on inaccurate availability or have difficulty reconciling a recorded sale. The model does not quantify whether or how often this occurs.

### 3.2 Catalogue changes can obscure historical facts

Product name and price can change, but a sale line stores copies of the values from the sale moment (§1, §§2.2, 2.4). The model treats those copied values as facts of the sale, not late reads from the current catalogue (§1). A product can also be recategorized; the recorded owner decision is to group report results by the frozen category label, even if one product consequently appears in more than one row (§11.1).

**Problem to solve:** allow current catalogue data to evolve without rewriting the historical meaning of earlier sales (§2.4, §11.1).

**Open conflict:** the owner decision in §11.1 conflicts with a “one row per product” criterion in the unavailable source specification. The model explicitly leaves that source criterion to be rewritten; CC-10 records the mismatch.

### 3.3 Invariants can be bypassed outside the application

The model distinguishes rules enforced by PostgreSQL from rules enforced only by C# (§§2, 4). It lists `price > 0`, `quantity > 0`, non-empty category name, valid role, and lowercase username as domain-only, with moving these checks to the engine tracked as T-20 (§2.2, §2.4, §2.5, §4). It also records pending or disputed physical-schema state in the consistency findings.

**Problem to solve:** make the location and strength of each rule explicit so that application behavior and stored data do not silently diverge (§4, §13). This challenge's documentation must preserve the distinction between an implemented safeguard, a domain-only rule, and planned work.

### 3.4 Identity and privacy need a clear boundary

The system distinguishes `admin` and `seller` users, stores a password hash, and attributes sales to internal operators (§1, §§2.3, 2.5, §7). The model says the domain never receives a clear-text password, and that hashes must not appear in logs, responses, projections, or errors (§2.5, §7, D-09). No buyer identity is modeled, and a seller breakdown is excluded from the report (DP-02, §7).

**Problem to solve:** support authentication and sale attribution without expanding the system into customer-data management or exposing authentication secrets (§7, §12).

## 4. People affected

| Party | Relationship to the problem | Evidence and limits |
|---|---|---|
| **Admin** | Manages seller accounts and is a modeled internal role. | Admin and seller roles are defined; the initial admin is provisioned at startup, and admin is not granted at runtime (§1, §2.5, §9.2, §11 H-3 / DP-04). The complete permissions for catalogue and reports are not specified. |
| **Seller** | Records sales under an internal identity. | Sale attribution and the seller role are modeled (§1, §§2.3, 2.5, §7). Exact permissions beyond user administration behavior are not fully specified. |
| **Deployment manager** | Supplies environment credentials used to provision the first admin. | Startup provisioning is described in §9.2. Treating deployment manager as a distinct person is **[assumption]**. |
| **Customer/buyer** | Not a modeled participant in the system. | No customer entity or buyer data exists in the model (§1, §7). |

## 5. Desired problem outcomes

These outcomes express the problem to resolve, not features or a full solution design:

1. **Valid stock:** no persisted product has negative stock, and a sale cannot withdraw more units than available (§2.2, §4).
2. **Consistent sale records:** a sale line and its stock withdrawal are treated as one business operation (§2.3). **[assumption]** The persistence implementation should make this operation atomic.
3. **Stable history:** later product edits do not change values already recorded on sale lines, and completed sales remain immutable (§1, §§2.3–2.4, §7.1).
4. **Understandable rule enforcement:** readers can distinguish database constraints from domain-only rules and pending tasks (§§2, 4, 13).
5. **Narrow data exposure:** internal identities are used only where the model requires them; password hashes and customer data are not exposed or introduced (§1, §7).
6. **Reliable historical reporting:** date-range reports are computed from sales and apply the frozen-label decision, with the source-spec conflict tracked rather than hidden (D-06, Q9 / §6.1, §11.1, CC-10).

**[assumption]** These outcomes are reasonable indicators that the product addresses the modeled problem. The source defines no quantitative targets for them.

## 6. Boundaries of this framing

The problem is limited to internal product catalogue, stock, user identity, sale recording, and a computed report (§§1–2, §6.1). It does not assert a specific interface, a current manual workflow, a business size, a deployment topology beyond what the model documents, or an integration with payment or customer systems (§7, §12).

The following are explicitly outside the modeled problem: customer/buyer accounts, multi-currency support, seller-level report breakdown, product attributes beyond the decided set, category CRUD, sale editing/deletion, physical product deletion, and `created_at` / `updated_at` audit columns (§1, §§2.1–2.3, §§7–8, §12, D-05, DP-02/03). Availability, backup/recovery, capacity, and numerical performance needs remain undefined; see NFR-13.

## 7. Assumptions and questions requiring evidence

| ID | Question or assumption | Why it matters | Status/source |
|---|---|---|---|
| PF-A1 | **[assumption]** The system serves a small internal stock-and-sales operation. | The model specifies internal roles and product/sale records but no organization size or operating context. | Inferred from §1 and §§2.3, 2.5; validate with owner. |
| PF-A2 | **[assumption]** Inconsistent stock or history would materially affect operators' work. | The model's safeguards imply these risks matter, but no impact evidence is included. | Inferred from §§2.2–2.4; no incident data supplied. |
| PF-A3 | **[assumption]** Sale persistence should use a transaction covering sale and stock updates. | The domain describes one operation, but the model does not spell out transaction boundaries or conflict recovery. | §2.3, D-04. |
| PF-Q1 | What exact permissions does each role have for catalogue operations and reports? | Roles exist, but a complete authorization matrix is not in the model. | User-story backlog S-1; **[assumption]** pending owner confirmation. |
| PF-Q2 | Will the source report criterion be updated to allow a row per product and frozen label? | The owner decision and source criterion conflict. | §11.1, CC-10. |
| PF-Q3 | Who owns orphan image cleanup? | The prescribed deletion order can leave an orphan binary; no cleanup process is defined. | §7.1, §11 H-2. |
| PF-Q4 | What operational targets are needed for availability, recovery, capacity, and response time? | None are established in the data model. | §12; NFR-13. |

## 8. Traceability summary

| Framing claim | Model evidence |
|---|---|
| Product and stock integrity problem | §1, §2.2, §4, ADR-002 |
| Sale/stock consistency problem | §2.3, D-04 |
| Historical value preservation | §1, §§2.3–2.4, §11.1 / CC-10 |
| Rule enforcement gaps | §§2, 4, 13; T-20 |
| Internal identity and privacy boundary | §1, §§2.3, 2.5, §7, §9.2, §11 H-3 / DP-04 |
| Report and data-scope limits | D-05/06, DP-02/03, §§1, 6.1, 8, 12 |