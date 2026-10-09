# Domain glossary

> **Purpose:** establish one shared vocabulary for the Simple Stock Flow domain. Definitions are reconstructed from §§0–1 of the [data model](../spec/data-model.md), then supplemented with entity rules in §2. Technical identifiers follow the model's naming convention. **[assumption]** marks any clarification not established by the model.

## 1. Catalogue and reference data

| Term | Domain definition | Technical representation and rules | Source |
|---|---|---|---|
| **Product** | An item offered in the catalogue. Its complete modeled attributes are name, price, stock, category, and optional image; no additional product attributes are included. | Product aggregate root; table `product`. Name is required and trimmed; price is positive; stock is nonnegative; category is required. | §1, §2.2, DP-03 |
| **Category** | A classification assigned to a product. The system has a fixed set of five seeded categories. | Category reference entity; table `category`. Read-only repository; no application operation creates, renames, or deletes a category. Name is unique in the database. | §2.1, §9.1, D-10 |
| **Price** | The current monetary amount for a product in the catalogue. It must be strictly positive. | `Money` value object; `product.price` is `numeric(18,2)`. The value object rounds to two decimal places using `MidpointRounding.AwayFromZero`. There is no currency column; the system is monocurrency. | §1, §2.2, §3, D-05 |
| **Stock** | The number of units available for a product. It must never be negative. | `product.stock`, integer. PostgreSQL enforces stock >= 0; Product behavior rejects a withdrawal greater than the available amount. | §1, §2.2, §4, ADR-002 |
| **Product image** | An optional image represented in the database only by an opaque key to a binary in external storage. It is neither the binary itself nor a path. | `product.image_key`; null means no image, never an empty string. Clear and commit the key before deleting the external binary. | §1, §2.2, D-08, §7.1 |
| **Soft deletion** | Removing a product from active catalogue use without physically deleting its database row. | `product.deleted_at` shadow property and a global filter; products with a deletion time are excluded from active reads. | §2.2, §3, ADR-003 |

## 2. Sales and historical values

| Term | Domain definition | Technical representation and rules | Source |
|---|---|---|---|
| **Sale** | An immutable completed commercial fact: who made it, when it happened, and what was sold. It records an internal operator; no buyer/customer is modeled. | Sale aggregate root; table `sale`. A sale must have at least one line before confirmation. No edit or delete port exists. | §1, §2.3, §7.1 |
| **Sale line / SaleItem** | A line within one sale, containing a product, quantity, and values copied at the time of sale. It has no independent lifecycle. | `SaleItem` entity inside the Sale aggregate; table `sale_item`. Its constructor is internal and it is created through `Sale.AddItem`. | §1, §2.4 |
| **Frozen value** | A copy of a value at the moment of sale. Later catalogue changes do not update that historical copy. | `sale_item.product_name` and `sale_item.unit_price`; a frozen category label is also specified by §2.4. The model's physical placement/status for `category_name` is disputed; see CC-02/CC-03. | §1, §2.4, consistency-check CC-02/CC-03 |
| **Quantity** | The number of units of one product recorded on a sale line. It must be strictly positive. | `Quantity` value object; `sale_item.quantity` is an integer. The positive-value rule is currently domain-only; T-20 is associated with moving it to the engine. | §1, §2.4, §4 |
| **Sale line subtotal** | The product of the frozen unit price and the line quantity. It is calculated, not stored as a column. | `SaleItem.Subtotal`; no database column. | §1 |
| **Sale total** | The sum of the sale-line subtotals. It is calculated, not stored as a column. | `Sale.Total`; no database column. | §1 |
| **Sale immutability** | Once registered, a sale is not edited or deleted through the modeled system. | No sale edit or delete port; sales and their lines are retained indefinitely. | §1, §2.3, §7.1 |

## 3. People, identity, and access

| Term | Domain definition | Technical representation and rules | Source |
|---|---|---|---|
| **User / operator** | An internal person who authenticates and can be identified as the operator recording a sale. The term does not mean a customer. | User aggregate root; table `user`. Username is required, unique, lowercase, and trimmed. | §1, §2.5 |
| **Admin** | One of the two permitted internal user roles. The initial administrator is provisioned by application startup; the role is not granted during normal runtime user creation. | `user.role = 'admin'`. The initial credentials come from the environment; the hash is produced through the hash port. | §1, §9.2, §11 H-3 / DP-04 |
| **Seller** | One of the two permitted internal user roles; a seller is created by an administrator and may be attributed to a sale. | `user.role = 'seller'`; current sale attribution is stored in `sale.sold_by`. The user foreign key is tracked as pending T-12. | §1, §2.3, §2.5, §3, §5 |
| **Role** | A closed set of internal access labels: `admin` or `seller`. | `user.role`. Membership in the set is currently enforced by domain code only; T-20 is associated with an engine constraint. The exact permission matrix is not defined in the model. | §1, §2.5, §4 |
| **Password hash** | An irreversible representation used for password verification. The clear-text password never enters the domain. | `user.password_hash`; produced through a hash port. It must never appear in logs, responses, projections, or error messages and is never indexed. | §1, §2.5, D-09, §7 |

## 4. Application concepts and read models

| Term | Domain definition | Technical representation and rules | Source |
|---|---|---|---|
| **Date range** | The time window used to request a sales report. Its end cannot be earlier than its start. | Application-layer value object; no database table. | §1 |
| **Sales report** | A calculated aggregation of sales by product for a date range. It is not persisted as its own entity. | Read model returned through a report read port; no report table. It is not broken down by seller (DP-02). | §1, §6.1, D-06, DP-02 |
| **Frozen category label in report** | The category label copied onto a sale line is used for report grouping. If a product's category changes, one product can appear in separate rows under different frozen labels. | Report query groups by product, product name, and category name, per the owner's decision. This conflicts with the source criterion “one row per product”; the conflict remains open. | §11.1, consistency-check CC-10 |

## 5. Naming and interpretation rules

- Table names are singular: `category`, `product`, `sale`, `sale_item`, and `user`. The database schema remains `sales`; for example, the sales aggregate table is `sales.sale` (§0).
- Database attributes use singular English `snake_case`; collection names in C# may be plural because they name object sets rather than tables (§0).
- A frozen value is historical data owned by the sale, not a late read of the current catalogue (§1).
- `user` is the table name in the `sales` schema. PostgreSQL's catalog may display it quoted; the model states that schema qualification means project queries can write `sales.user` without quoting (§0).
- “User” in this glossary means an internal operator. “Customer” or “buyer” is not a synonym and is outside the modeled domain (§1, §7).

## 6. Terms deliberately not included

The following concepts are absent or explicitly excluded from the model; they must not be introduced as existing domain entities without a new decision:

| Excluded term/concept | Reason | Source |
|---|---|---|
| **Customer / buyer** | No customer entity or buyer data; the sale records its internal operator. | §1, §7 |
| **Sale currency** | The system is monocurrency and has no currency column. | D-05, §3, §12 |
| **Product SKU, description, or reference code** | Product attributes are closed to name, price, stock, category, and optional image. | DP-03, §1, §12 |
| **Seller report breakdown** | The report is not grouped by seller, and that scope is explicitly closed. | DP-02, §6.3, §7.1 |
| **Persisted report** | The report is calculated when requested; there is no report table. | §1, D-06 |
| **Catalogue audit timestamps** | `created_at` and `updated_at` columns are explicitly not part of the system. | §8 |

## 7. Related domain documents

- [Entities and rules](entities-and-rules.md) describes aggregate boundaries and where each invariant is enforced.
- [Domain events](domain-events.md) lists candidate event names as **[assumption]** only; the model does not require event publication, a broker, or an outbox.
- [Context glossary](../01-context/glossary.md) provides the shorter vocabulary used by the system-context document.