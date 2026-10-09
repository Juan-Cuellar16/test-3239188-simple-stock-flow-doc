# Domain entities and rules

> **Traceability:** Reconstructed from the [data model](../spec/data-model.md). Every entity, relationship, and invariant below cites its model evidence. The data model contains contradictory statements about some physical schema state; those are called out using the findings in [the architecture consistency check](../05-architecture/consistency-check.md), especially CC-02–CC-07. This document does not treat a disputed measurement as settled fact.

## 1. Domain model at a glance

The model contains five entities grouped into **three aggregate roots**: Product, Sale, and User. Category is reference data, not an aggregate root; SaleItem is an entity inside the Sale aggregate and cannot exist independently (§2.1–2.5). The relationships are Category → Product, Sale → SaleItem, Product → SaleItem, and User → Sale (§2, §5).

| Aggregate / entity | Boundary | Persistence representation | Primary responsibility |
|---|---|---|---|
| **Product** | Aggregate root; owns catalogue state and stock behavior. | `product` | Maintain the product's allowed attributes, category association, stock, image key, and active/removed state (§2.2). |
| **Sale** | Aggregate root; owns its SaleItems. | `sale` and `sale_item` | Record an immutable completed sale and coordinate line creation with stock withdrawal (§2.3–2.4). |
| **User** | Aggregate root. | `user` | Represent an internal operator, normalized username, password hash, and role (§2.5). |
| **Category** | Reference entity; not an aggregate root. | `category` | Supply one of five fixed, seeded classifications to products (§2.1, §9.1). |
| **SaleItem** | Internal entity in Sale; constructed only through `Sale.AddItem`. | `sale_item` | Record a product, quantity, and frozen values for one sale line (§2.4). |

## 2. Entities, invariants, and lifecycle

### 2.1 Category — fixed reference data

Category identifies a product classification. Its name is required, non-empty, and trimmed by domain behavior; a unique database index enforces name uniqueness (§2.1). The category repository is read-only: there is no application operation to create, rename, or delete categories. The five category rows are inserted by the initial migration (§2.1, §9.1, D-10).

Uniqueness is sensitive to case and accents: values such as “Fontanería” and “Fontaneria” can be distinct. This is accepted while category data is fixed and read-only; if category maintenance is introduced, the model says to revisit the choice before implementing it (§4.1).

### 2.2 Product — catalogue aggregate

Product is the aggregate root for catalogue data and stock behavior (§2.2). Its modeled attributes are name, price, stock, category, and optional image only (DP-03, §1).

| Rule | Enforcement and lifecycle |
|---|---|
| Name is required, non-empty, and trimmed. | `NOT NULL` is in the engine; non-empty validation and trimming are in `Product.Rename` (§2.2). |
| Price is strictly positive. | `Product.ChangePrice` enforces it in the domain. `Money` accepts zero; the database check is tracked as T-20 (§2.2, §4). |
| Stock cannot be negative. | The database enforces `ck_product_stock_non_negative`. `Product.Withdraw` / `Restock` implement stock operations (§2.2, §4, ADR-002). |
| A withdrawal cannot exceed available stock. | Domain process rule in `Product.Withdraw`; it is not expressible as a simple `CHECK` (§2.2). |
| Category is required and must exist. | Product behavior and FK-1 (`RESTRICT`) (§2.2, §5). |
| Image is optional. | `image_key` is nullable; absence is represented by `NULL`, not an empty string. The binary itself is external and addressed by an opaque key (D-08, §2.2, §7.1). |
| Removal is logical, not physical. | `deleted_at` plus the persistence filter represent soft deletion; the later debt register says T-09 is applied. Older prose conflicts, so see CC-05 (§2.2, §3, §13 D-1). |

Money is rounded to two decimal places using `MidpointRounding.AwayFromZero`; the database column is `numeric(18,2)`. A change to either precision must be coordinated (§2.2). Optimistic concurrency uses PostgreSQL `xmin` (D-04, §3, ADR-002).

### 2.3 Sale — sales aggregate

Sale is the aggregate root for a completed commercial fact: operator, sale time, and sale lines (§1, §2.3). Its rules are:

- The operator is required and non-empty in domain behavior; `sold_by` is non-null in the model's current schema (§2.3, §3).
- A sale must have at least one line before it can be confirmed. `Sale.EnsureConfirmable` enforces this; a database `CHECK` cannot express the cross-row condition (§2.3).
- A product cannot occur more than once in the same sale. `Sale.AddItem` rejects duplicates; §13 D-2 says a unique `(sale_id, product_id)` index was added in T-20. CC-04 records a conflicting older index inventory, so current engine state should be read with that finding in view (§2.3, §4, §13 D-2, CC-04).
- Adding a line and withdrawing stock are one domain operation: `Sale.AddItem` calls `Product.Withdraw` before adding the line (§2.3).
- A registered sale cannot be edited or deleted through the modeled application; it is retained indefinitely (§1, §2.3, §7.1).

The database relationship from Sale to SaleItem is composition: FK-2 cascades from sale to its lines, while the sale line's `sale_id` is required in the later T-20 status (§2.4, §5, §13 D-2). The cascade is a structural relationship; the model says sales themselves are never deleted (§7.1).

### 2.4 SaleItem — internal sale-line entity

SaleItem is owned by Sale and cannot be created independently. Its constructor is internal and `Sale.AddItem` is the only legitimate creation path (§2.4).

| Rule | Enforcement and lifecycle |
|---|---|
| A product reference is required. | `product_id` is non-null; the later status in §13 D-2 says FK-3 (`RESTRICT`) exists from T-20. Older sections conflict; see CC-05. |
| Quantity must be positive. | Enforced by the `Quantity` value object in the domain; the engine check is associated with T-20 (§2.4, §4). |
| Product name and unit price are frozen at sale time. | `Sale.AddItem` copies these values from Product; later catalogue changes do not rewrite them (§1, §2.4). |
| Category label is frozen at sale time. | §2.4 specifies `sale_item.category_name` without a category FK. CC-02 resolves the conflicting §3 Product-column text as a copy/paste error; CC-03 says engine/pending status is uncertain. |
| It cannot exist without its sale. | FK-2 uses `ON DELETE CASCADE`; the later model status says `sale_id` is `NOT NULL` (§2.4, §5, §13 D-2; CC-05). |

The frozen category label is used by the report. If a product changes category within the report range, the report can show the product under multiple frozen labels (§11.1). The owner decision conflicts with the original “one row per product” source criterion; that source change remains open (CC-10).

### 2.5 User — identity aggregate

User represents an internal operator, not a customer or buyer (§1, §2.5, §7). It has a required unique username, a required password hash, and one of two roles: `admin` or `seller` (§1, §2.5).

- `User.NormalizeUsername` trims and lowercases the username; this is domain-only today (§2.5, §4).
- The username is unique in the engine through `IX_user_username`; lowercase normalization is not enforced by the current database rule (§2.5, §4).
- `password_hash` is required. The domain never receives the clear-text password; a hash port produces it (D-09, §2.5, §7).
- `Roles.IsValid` accepts only `admin` and `seller`; enforcement is domain-only, with an engine check tracked as T-20 (§2.5, §4).
- The initial admin is created during application startup using environment credentials, not seeded as a database hash (§9.2). Admin is not granted at runtime; admins create sellers (DP-04, §11 H-3).

The current `sale.sold_by` value is text. A `sold_by_user_id` reference to User is pending T-12 (FK-4), so the model does not yet guarantee the relational attribution by foreign key (§3, §5).

## 3. Value objects and derived concepts

| Concept | Rule | Persistence |
|---|---|---|
| **Money** | Rounds to two decimals using `AwayFromZero`; Product price must be > 0. The sale aggregate has a currency guard as tracked debt T-05 (§2.2–2.3). | Stored as `numeric(18,2)`; no currency column (D-05, §3). |
| **Quantity** | Must be strictly positive (§2.4). | `sale_item.quantity`, integer. |
| **Date range** | The report end must not be earlier than the start (§1). | Application-layer value object; no table. |
| **Sale subtotal / total** | Subtotal is unit price × quantity; sale total sums subtotals (§1). | Calculated in the domain; no column. |
| **Sales report** | Aggregation by product over a date range; calculated in the database read path and not persisted (D-06, Q9 / §6.1). | Read model; no table. Not broken down by seller (DP-02). |

## 4. Relationships and deletion behavior

| Relationship | Integrity rule | Delete behavior / current status |
|---|---|---|
| Product → Category | FK-1 from `product.category_id` to `category.id` (§5). | `RESTRICT`; a category referenced by products cannot be deleted. Category deletion is not exposed by an application port (§2.1, §5). |
| Sale → SaleItem | FK-2 from `sale_item.sale_id` to `sale.id` (§5). | `CASCADE`; a line belongs to its sale. Sales are retained and have no delete operation (§2.3, §5, §7.1). |
| SaleItem → Product | FK-3 from `sale_item.product_id` to `product.id` (§5). | `RESTRICT`; §13 D-2 says implemented in T-20, though older prose is stale (CC-05). Product removal is soft delete (§2.2, ADR-003). |
| Sale → User | FK-4 from `sale.sold_by_user_id` to `user.id` (§5). | `RESTRICT`, pending T-12. Current sale attribution is text in `sold_by` (§3, §5). |

## 5. Rule-enforcement summary

The model distinguishes three states: **engine**, **domain-only**, and **pending**. A rule that lives only in C# can be bypassed by a direct database write; pending work is not an implemented guarantee (model header, §4, §13).

| Invariant | Current classification in the model | Reference / caveat |
|---|---|---|
| Primary keys for the five tables | Engine | §4 |
| Category name and username are unique | Engine (unique indexes) | §2.1, §2.5, §4 |
| Product stock is nonnegative | Engine (`ck_product_stock_non_negative`) | §2.2, §4 |
| Product price is positive | Domain-only; T-20 check planned | §2.2, §4 |
| Product name and category name are non-empty | Domain-only; T-20 checks planned | §§2.1–2.2, §4 |
| Withdrawal cannot exceed available stock | Domain-only process rule | §2.2 |
| Product category exists | Engine FK-1 and domain behavior | §2.2, §5 |
| Sale contains at least one line before confirmation | Domain-only; would require a deferred trigger to enforce in the database | §2.3 |
| Sale has at most one line per product | Domain plus unique composite index according to §13 D-2; older inventory conflicts | §2.3, §4, §13 D-2, CC-04 |
| Sale-line quantity is positive | Domain-only; T-20 check planned | §2.4, §4 |
| Sale line belongs to an existing sale | Engine FK-2; required `sale_id` according to later status | §2.4, §5, §13 D-2, CC-05 |
| Sale line references an existing product | Engine FK-3 according to later status; older prose says otherwise | §5, §13 D-2, CC-05 |
| Username is lowercase and role is valid | Domain-only; T-20 checks planned | §2.5, §4 |
| Sale attribution points to a User row | Pending T-12 / FK-4 | §3, §5 |

## 6. Domain boundaries and exclusions

- The report is a read model, not an aggregate or persisted entity (§1, D-06).
- Product image binaries are outside the domain database; only their opaque keys are stored (D-08, §7.1).
- Customer identity, multi-currency amounts, seller-report breakdown, and extra product attributes are excluded by the model (D-05, DP-02/03, §§1, 12).
- No event publication, broker, or outbox is specified; candidate event names are **[assumption]** and documented separately in [domain-events.md](domain-events.md) (§2, §6, §12).
- The model's physical column count and some schema inventories are inconsistent; use CC-01–CC-07 and the model's current query outputs before claiming a disputed physical fact.
