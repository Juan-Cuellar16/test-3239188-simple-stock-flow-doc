# Domain events

> **Traceability and status:** The [data model](../spec/data-model.md) defines domain entities, operations, and state changes, but it does **not** define a domain-event catalogue, an event contract, a broker, an outbox, or asynchronous consumers (§2, §6, §12). Every event name and event boundary in this document is therefore a **[assumption]** proposed as vocabulary for describing meaningful domain facts. These proposals do not claim that the current system emits or persists events.

## 1. Purpose and terminology

A domain event is a statement that something meaningful in the domain has happened. In this document, candidate events help name important outcomes of Product, Sale, and User behavior (§2.2–2.5). They are not additional entities, tables, requirements, or integration messages.

The model supports synchronous access through repositories and a report read port; it does not describe asynchronous communication (§6.1, §12). **[assumption]** If the application later needs these events, begin with in-process domain notifications and introduce durable or external publication only when a concrete consumer and delivery requirement are identified.

## 2. Candidate event catalogue

Every row below is a proposal, not an implemented contract. Payload fields are intentionally not specified: the model defines no serialized event schema (§12).

| Candidate event | Occurrence in domain terms | Model evidence | Status and cautions |
|---|---|---|---|
| **ProductCreated** | A new valid Product becomes part of the catalogue. | Product aggregate and its allowed attributes (§1, §2.2, DP-03). | **[assumption]** The model describes Product rules but does not name a create operation or event. |
| **ProductRenamed** | The active Product's name is changed. | `Product.Rename`; sale-line name remains frozen (§1, §2.2, §2.4). | **[assumption]** Event name and publication are not specified. |
| **ProductPriceChanged** | The active Product receives a new positive catalogue price. | `Product.ChangePrice`; the price must be > 0 (§2.2, §4). | **[assumption]** Do not use this event to rewrite existing sale lines (§2.4). |
| **ProductRestocked** | Stock is increased through Product behavior. | `Product.Restock`; stock remains nonnegative (§2.2, §4). | **[assumption]** The model does not prescribe an event or a restock audit record. |
| **ProductCategoryChanged** | A Product is assigned to a different existing category. | Product category is required and linked by FK-1 (§2.2, §5). | **[assumption]** The operation is implied by `Product.SetCategory`; the event name and consumers are not defined. |
| **ProductSoftDeleted** | A Product is removed from active catalogue reads by setting its deletion state. | `deleted_at` and the global filter; no physical delete (§2.2, §3, ADR-003). | **[assumption]** This should not mean a physical row deletion. Current T-09 status has stale prose; see CC-05. |
| **ProductImageAttached** | An opaque key is associated with a Product after an external binary is stored. | Image key and external storage behavior (D-08, §1, §7.1). | **[assumption]** Storage succeeds before the key is persisted; no event or exact failure handling is specified. |
| **ProductImageDetached** | A Product's image key is cleared and committed before the external binary is deleted. | Required deletion order and lack of cross-resource atomicity (§7.1). | **[assumption]** This name describes the database-side change. A failed binary delete may leave an orphan; cleanup is unresolved (§11 H-2). |
| **SaleRegistered** | A Sale is confirmed with at least one line after the corresponding stock withdrawals succeed. | `Sale.EnsureConfirmable`, `Sale.AddItem`, Sale immutability (§2.3–2.4). | **[assumption]** The model defines the business operation but not an emitted event or transaction/publication guarantee. |
| **SellerCreated** | A new User with role `seller` is created by an administrator. | User roles, seller creation, no runtime admin grant (§2.5, §11 H-3 / DP-04). | **[assumption]** User creation has no named event. Avoid including password or password hash in any event; see §7. |
| **InitialAdminProvisioned** | Startup provisions the first admin from environment credentials. | Startup seed path and hash port (§9.2, D-09, D-10). | **[assumption]** The model describes provisioning, not a lifecycle event. It must not carry credentials or hash material. |

### Events intentionally not proposed

- **CategoryCreated / CategoryRenamed / CategoryDeleted:** the five categories are fixed reference data; the repository is read-only (§2.1, §9.1).
- **SaleEdited / SaleDeleted:** registered sales are immutable and retained (§1, §2.3, §7.1).
- **ReportGenerated / ReportStored:** the report is a computed read model, not a persisted entity (§1, D-06, §6.1).
- **CustomerCreated / CustomerPurchased:** the model has no customer or buyer concept (§1, §7).

## 3. Aggregate ownership and candidate timing

The model defines Product, Sale, and User as aggregate roots; Category is not a root, and SaleItem belongs inside Sale (§2.1–2.5). **[assumption]** If candidate events are adopted, the aggregate that owns the change should originate the event:

| Aggregate / boundary | Candidate events it could originate | Timing guidance (proposed) |
|---|---|---|
| Product | ProductCreated, ProductRenamed, ProductPriceChanged, ProductRestocked, ProductCategoryChanged, ProductSoftDeleted, ProductImageAttached, ProductImageDetached | **[assumption]** Raise an event only after the aggregate accepts the state change; persist the resulting state before treating it as an externally observable fact. |
| Sale (including SaleItem) | SaleRegistered | **[assumption]** Do not publish a registered-sale event if confirmation fails or if stock and sale persistence do not succeed together. The model requires the domain operation to coordinate line creation and stock withdrawal (§2.3); transaction mechanics are unspecified. |
| User | SellerCreated, InitialAdminProvisioned | **[assumption]** Raise only after the account is created successfully; no password or password hash belongs in event data (§7, §9.2). |
| Category | None proposed | Category data is seeded and read-only (§2.1, §9.1). |

SaleItem is not assigned a separate event owner because it has no independent lifecycle; meaningful sale-line facts belong to its Sale aggregate (§2.4).

## 4. Delivery and consistency boundaries

The model does not require event persistence or delivery. It describes database transactions and external image storage only as far as the image key ordering is concerned (§7.1, §12).

- **[assumption] In-process events:** If a use case needs to notify another part of the same application, an in-process notification could be considered. The model does not require handlers or prescribe their execution order.
- **[assumption] External messages:** Do not claim at-least-once, exactly-once, or guaranteed delivery. There is no broker, outbox, message schema, retry policy, or consumer contract in the model (§2, §6, §12).
- **[assumption] Database commit boundary:** A candidate event that describes persisted state should not be treated as an accomplished fact before the corresponding state is committed. The model does not specify an outbox or a reliable way to atomically publish an event.
- **Image storage:** Database and binary deletion are not atomic. Clear and commit `image_key` first, then delete the binary; an orphan binary is possible and no cleanup process is defined (§7.1, §11 H-2). A ProductImageDetached notification cannot by itself promise that binary cleanup completed.
- **Report generation:** The report is computed through a read port, so a report result is not a domain event (§1, D-06, §6.1).

## 5. Event data and privacy cautions

The model classifies username and seller identity as personal data, role as internal-confidential, and `password_hash` as an authentication secret. The hash must never appear in logs, responses, projections, or errors (§7). **[assumption]** Apply the same minimization to any future event payload:

1. Never include a clear-text password or `password_hash` (D-09, §7).
2. Include seller identity only if a consumer has a defined need; the report is explicitly not broken down by seller (DP-02, §7.1).
3. Prefer identifiers and business facts needed by a consumer over copying entire database rows. The model does not define event payloads.
4. Preserve the distinction between frozen sale values and current Product values; a future event must not imply that a historical sale follows catalogue changes (§1, §2.4).

## 6. Ordering, retries, and duplicate handling

The data model specifies no event ordering, retry, deduplication, idempotency, replay, or retention policy (§12). These policies must not be described as current guarantees.

**[assumption]** If asynchronous publication is later introduced, the owner and architecture should define at least:

- Whether events are delivered in aggregate order and how that order is represented.
- Whether consumers may receive a duplicate and how they make processing idempotent.
- What happens when publication fails after the database transaction commits.
- How event data is retained, redacted, and made available for replay.
- How image cleanup failures are surfaced and retried, given the orphan-binary case (§7.1, §11 H-2).

## 7. Decisions required before implementing event publication

| Decision | Why it is open | Evidence |
|---|---|---|
| Are domain events required, and who consumes them? | No event consumers or asynchronous use cases are described. | §6.1, §12 |
| Are these in-process notifications or integration messages? | The model describes neither delivery mechanism nor message boundary. | §2, §6, §12 |
| Is durable delivery required? | No outbox, event store, retry policy, or delivery guarantee is specified. | §12 |
| Which event data may include personal identifiers? | Username and seller identity are personal data; no event payload policy is defined. | §7 |
| Who cleans up orphan images? | The prescribed order can leave an orphan binary after a failed deletion. | §7.1, §11 H-2 |

Until these decisions are made, candidate names in this document are vocabulary proposals only. They do not expand product scope or contradict the architecture overview's statement that asynchronous messaging is not defined.

## 8. Traceability summary

| Event-related statement | Model evidence |
|---|---|
| Aggregate ownership and SaleItem lifecycle | §2.1–2.5 |
| Stock and sale operation | §2.2–2.3, §4, D-04 |
| Frozen sale values and report grouping | §1, §2.4, §11.1 / CC-10 |
| Fixed category lifecycle | §2.1, §9.1 |
| Internal identity and secret handling | §1, §2.5, §7, §9.2 |
| Image ordering and orphan risk | D-08, §7.1, §11 H-2 |
| No event-delivery architecture specified | §6.1, §12 |
