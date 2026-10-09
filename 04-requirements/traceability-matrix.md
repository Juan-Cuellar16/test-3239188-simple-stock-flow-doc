# Requirements traceability matrix

> Stories are reconstructed in [user-stories.md](user-stories.md). This matrix maps each story to its model evidence and the architectural capability/access pattern that must support it. A reference to a query pattern indicates a read capability, not an API contract. Unspecified behavior remains marked **[assumption]** in the source story.

| Story | Capability / model evidence | Data-model rules or patterns | Architecture mapping |
|---|---|---|---|
| HU-001 Log in | User lookup, password verification and role identity (§2.5, §7, D-09) | Q10, unique IX_user_username (§§6.1–6.2) | User repository + password-hash port; authentication transport is **[assumption]** |
| HU-002 Provision first administrator | Environment-based first-admin creation (§9.2; §10.4) | Hash port; do not seed a hash literal (D-09, D-10) | Startup/application provisioning + hash adapter |
| HU-003 Create sellers | Create User; closed roles; no runtime admin grant (§2.5, §11 H-3, §13 D-3) | Username uniqueness; normalization is domain-only (§4) | User write port + authorization boundary |
| HU-004 List categories | Fixed read-only category reference data (§2.1, §9.1) | Five seeded rows; unique name (§2.1, §9.1) | Read-only category repository |
| HU-005 Create product | Product aggregate with valid name, price, stock, category (§2.2) | FK-1; stock check; price > 0 domain-only (§§4–5) | Product repository + category lookup |
| HU-006 Search/open products | Search by name/category, get by ID (Q1–Q3) | Soft-delete filter and planned search indexes (§2.2, §§6.1–6.2, ADR-003) | Product read port; persistence adapter |
| HU-007 Rename/reprice product | Change catalog values while preserving sale snapshots (§2.2, §1) | Frozen sale name/price (§2.4) | Product aggregate/repository |
| HU-008 Restock product | Increase stock; maintain nonnegative invariant (§2.2) | ck_product_stock_non_negative; concurrency D-04 / ADR-002 | Product aggregate + optimistic-concurrency persistence |
| HU-009 Manage image | Store opaque key and coordinate external binary lifecycle (D-08, §7.1) | Nullable image_key; clear key before external deletion | Product repository + image-storage port |
| HU-010 Soft-remove product | Remove from active catalogue without physical deletion (§2.2, ADR-003) | deleted_at global filter; FK-3 restrict protects history (§5) | Product repository soft-delete operation |
| HU-011 Register sale | Create immutable sale and lines; withdraw stock (§§2.3–2.4) | FK-2; one product per sale; sale/stock operation (§§2.3–2.4, §5) | Sale and product aggregates coordinated by application service |
| HU-012 Reject unfulfillable sale | Reject invalid quantity, missing product, or insufficient stock (§§2.2–2.4) | quantity > 0 domain-only; stock check is domain-only; product FK-3 (§§4–5) | Domain validation + persistence constraint |
| HU-013 Freeze sold values | Preserve product name, price, and category label at sale time (§1, §2.4) | Snapshot fields on sale_item; category label state has CC-03 caveat | Sale aggregate and snapshot persistence |
| HU-014 View a sale | Read sale and its lines (Q6) | Sale/line composition and retention (§§2.3–2.4, §7.1) | Sale read port |
| HU-015 List sales by date range | Query sales in an interval (Q7) | sold_at, index IX_sale_sold_at (§§3, 6.1–6.2) | Sale query port |
| HU-016 Sales report | Aggregate sold quantities and amounts for a date range; order by amount descending; do not break down by seller (Q9, D-06, DP-02) | Report is computed, not persisted; category label grouping follows the owner decision in §11.1 | Report read port/query adapter; output columns are **[assumption]** (see user story) |
| HU-017 Stable closed report | Preserve frozen product name and category label; recategorization may produce multiple rows for one product (§11.1) | Group by product, product name, and frozen category name. CC-10 records the conflict with the source spec's “one row per product” criterion | Report read port/query adapter; follows owner decision pending the source-criterion rewrite |

## Cross-cutting quality traceability

| NFR | Supporting model evidence | Architecture concern |
|---|---|---|
| NFR-01 Integrity | §2.2, §4, ADR-002 | Domain invariants + database constraint |
| NFR-02 Atomic sale | §2.3 | Application coordination; transaction policy is **[assumption]** |
| NFR-03 Concurrency | D-04, ADR-002, §3 | Persistence concurrency token |
| NFR-04–06 Security | D-09, §2.5, §7, §9.2, §13 D-3 | Hash and identity ports; authorization boundary |
| NFR-07 History | §§1–2.4, §7.1, ADR-003 | Immutable sale model + soft deletion |
| NFR-08 Images | D-08, §7.1 | External storage port; ordered deletion |
| NFR-09–10 Precision/time | D-05, §§2.2–3 | Value-object mapping and timestamp persistence |
| NFR-11 Query performance | §§6.1–6.3, T-13 | Query-specific indexes; no latency SLO implied |
| NFR-12 Data minimization | DP-02/03/05, §§1, 8, 12 | Scope control in schema and ports |
| NFR-13 Operational objectives | Model silence | **[assumption]** Owner must define availability/recovery/capacity goals |

### Traceability limits

- The model does not define HTTP routes, request/response shapes, error messages, or token format (§12); these are not inferred as facts here.
- The model contains contradictory or stale claims identified as CC-01–CC-10 in [consistency-check.md](../05-architecture/consistency-check.md). For disputed implementation state, keep the finding visible rather than silently inventing certainty.
