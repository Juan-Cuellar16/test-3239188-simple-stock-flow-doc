# User stories

> Traceability: cite the data model on every statement (section 2.3, FK-2, D-04). Mark anything that is not in the model as **[assumption]**.

> **What this is:** the User Story backlog of Simple Stock Flow, rebuilt **backwards from the data model**
> ([`../spec/data-model.md`](../spec/data-model.md)). The model lists no requirements, but every table, rule and constraint exists
> because some story needs it; each story below points to the part of the model that makes it necessary.
> **How to cite:** `§2.3` = section of the data model · `Q9` = access pattern (§6.1) · `FK-2` = foreign-key policy (§5) ·
> `D-04` / `DP-02` / `ADR-002` = decisions the model mentions · `T-xx` = tasks the model mentions.
> **Priorities (MoSCoW), story points and sprints are not in the model.** Priorities are an **[assumption]** of this document; estimation is left open.
> **Error messages and HTTP codes are not defined in the model** (they live in `api-contract.md`, not delivered). Scenarios therefore say
> "is rejected and nothing is saved", never a message text.

---

## Backlog status

| Cut | Priority | Sprint | Total HUs | Refined | In progress | Completed |
|-----|----------|--------|-----------|---------|-------------|-----------|
| Cut 1 | Must Have | To be planned | 10 | 0 | 0 | 0 |
| Cut 2 | Should Have and Could Have | To be planned | 7 | 0 | 0 | 0 |

> A story counts as *Refined* when its acceptance criteria are written **and** its story points are estimated. The criteria are written; the estimation is still pending.

---

## Actors

| Actor | Source |
|---|---|
| **Administrator** (`admin`) | §1 *Rol*; provisioned by the deployment, never granted at runtime (DP-04, §11 H-3) |
| **Seller** (`seller`) | §1 *Rol*; created by an administrator (§11 H-3) |
| **Deployment manager** | Supplies the first administrator's credentials from the environment (§9.2) · **[assumption]** that this is a separate actor |
| *Customer / buyer* | **Does not exist** (§1 *Usuario*, §7) |

> **[assumption] S-1:** the model does not say which role may do each operation. This backlog assumes that **both roles read**, that **only `admin` changes the catalogue and reads the report**, and that **sellers register sales**. To be confirmed by the owner.

## Epics

| ID | Epic | Description |
|----|------|-------------|
| EP-001 | Identity and access | Log in, first administrator, creating sellers (§2.5, §9.2, §11 H-3) |
| EP-002 | Catalogue | Categories, products, stock, images and soft removal (§2.1, §2.2, ADR-003) |
| EP-003 | Sales | Registering immutable sales that move stock and freeze values (§2.3, §2.4) |
| EP-004 | Sales report | Aggregated, stable report by product (Q9, D-06, §11.1) |

---

## User stories

### HU-001 — Log in {#HU-001}

**Epic:** EP-001
**Data model references:** Q10 (§6.1), §2.5, §7, D-09

> **As** an operator (administrator or seller)
> **I want** to log in with my username and password
> **so that** I work under my own identity and only with the functions of my role

**Acceptance Criteria:**

```gherkin
Scenario 1: Successful login
  Given an existing user and its correct password
  When  I log in with that username and password
  Then  I receive proof of identity that carries my username and role
  # the proof is a token [assumption]; the model only shows 401 without a token (§13 D-3)

Scenario 2: Wrong password
  Given an existing user
  When  I log in with an incorrect password
  Then  access is denied and no proof of identity is issued

Scenario 3: Unknown username
  Given a username that does not exist
  When  I try to log in
  Then  access is denied and no proof of identity is issued

Scenario 4: Username typed with capitals or spaces
  Given a user stored as "ana"
  When  I log in typing " Ana "
  Then  I am identified as "ana"
  # normalisation before lookup is an [assumption]; the model says usernames are stored lower-case and trimmed (§2.5)

Scenario 5: Secrets are never shown
  Given any login attempt, successful or not
  When  the response and the logs are inspected
  Then  neither the password nor the stored hash appears in them (§7)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-002 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** the lookup is by exact username and happens on every login, so it is a high-frequency query served by the unique index `IX_user_username` (Q10, §6.2).

---

### HU-002 — Provision the first administrator at startup {#HU-002}

**Epic:** EP-001
**Data model references:** §9.2, D-09, D-10, §10.4

> **As** the deployment manager
> **I want** the application to create the first administrator when it starts, from credentials in the environment
> **so that** the system has a first administrator without any credential being stored in the repository

**Acceptance Criteria:**

```gherkin
Scenario 1: Empty system
  Given the table user has no rows and the administrator credentials are set in the environment
  When  the application starts
  Then  one user with role admin exists
  And   its password_hash was produced by the hash port, never the plain password

Scenario 2: Nothing versioned
  Given the repository and the migrations
  When  they are searched for a password or a hash literal
  Then  none is found
  # the seed is not done in SQL: it would need a precomputed hash versioned in the repository (§9.2, article IX)

Scenario 3: Restart
  Given the administrator already exists
  When  the application starts again
  Then  no second user is created
  # idempotent restart is an [assumption]; the model only guarantees unique usernames (§4)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | None |
| Affected service(s) | simple-stock-flow-api |

> **Note:** the model measured the result of this story: the table `user` holds exactly one row (§10.4).

---

### HU-003 — Create sellers {#HU-003}

**Epic:** EP-001
**Data model references:** §2.5, §11 H-3 (DP-04), §13 D-3, §9.2

> **As** an administrator
> **I want** to create seller accounts
> **so that** every sale is attributed to the person who made it (§2.3, §7)

**Acceptance Criteria:**

```gherkin
Scenario 1: Create a seller
  Given I am logged in as an administrator
  When  I create a user with a new username, a password and role seller
  Then  the user exists with role seller
  And   the username is stored in lower case and without surrounding spaces
  And   only the password hash is stored, never the password

Scenario 2: Username already taken
  Given a user "ana" exists
  When  I create a user typed " Ana "
  Then  the creation is rejected and no user is added
  # without normalisation, "Ana " would become an account that can never log in (§2.5)

Scenario 3: No token
  Given a request without proof of identity
  When  it tries to create a user
  Then  the system answers 401 and creates nothing (§13 D-3)

Scenario 4: Seller tries
  Given I am logged in as a seller
  When  I try to create a user
  Then  the system answers 403 and creates nothing (§13 D-3)

Scenario 5: Nobody grants admin at runtime
  Given I am logged in as an administrator
  When  I try to create a user with role admin
  Then  it is rejected: the admin role is provisioned by the deployment (DP-04)
  # the exact response is not defined in the model

Scenario 6: Role outside the closed set
  Given I am logged in as an administrator
  When  I create a user with a role that is neither admin nor seller
  Then  the creation is rejected (§2.5, Roles.IsValid)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-001, HU-002 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** defect A-1 (anonymous user creation) was **closed**; Scenarios 3 and 4 are its regression guard (§13 D-3). Today `role` and lower-case are guaranteed by C# only; T-20 moves them to the engine (§4).

---

### HU-004 — List the categories {#HU-004}

**Epic:** EP-002
**Data model references:** Q4, Q5 (§6.1), §2.1, §9.1, D-10

> **As** an administrator
> **I want** to see the categories
> **so that** I can classify each product and filter searches by category

**Acceptance Criteria:**

```gherkin
Scenario 1: Five fixed categories
  Given a freshly migrated system
  When  I list the categories
  Then  I see exactly five, ordered by name: Electricidad, Fontanería, General, Herramientas, Pinturas (§9.1)

Scenario 2: Read-only
  Given any role
  When  I look for an operation to create, rename or delete a category
  Then  none exists (§2.1)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Should Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-001 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** without seeded categories no product can be created, because the category is mandatory (FK-1, §9.1).

---

### HU-005 — Create a product {#HU-005}

**Epic:** EP-002
**Data model references:** §2.2, §1 (*Producto*, *Precio*, *Stock*), DP-03, FK-1, D-05

> **As** an administrator **[assumption] S-1**
> **I want** to register a product with name, price, stock and category
> **so that** it can be found and sold

**Acceptance Criteria:**

```gherkin
Scenario 1: Valid product
  Given a name, a price greater than zero, a stock of zero or more and an existing category
  When  I create the product
  Then  it is saved with the name trimmed
  And   it has exactly five attributes: name, price, stock, category and optional image (DP-03)

Scenario 2: Empty name
  Given a name that is empty or only spaces
  When  I create the product
  Then  it is rejected and nothing is saved

Scenario 3: Price not positive
  Given a price of 0 or less
  When  I create the product
  Then  it is rejected and nothing is saved

Scenario 4: Negative stock
  Given a stock below zero
  When  I create the product
  Then  it is rejected and nothing is saved

Scenario 5: Category does not exist
  Given a category identifier that is not one of the five
  When  I create the product
  Then  it is rejected and nothing is saved (FK-1)

Scenario 6: Price rounding
  Given a price of 10.005
  When  I create the product
  Then  the stored price is 10.01 (two decimals, AwayFromZero; §2.2)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-001, HU-004 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** `price > 0` is guaranteed by C# only; `Money` itself accepts zero. T-20 moves the rule to the engine (§2.2, §4). There is no currency in the story: single currency (D-05).

---

### HU-006 — Search and open products {#HU-006}

**Epic:** EP-002
**Data model references:** Q1, Q2 (§6.1), §6.2, ADR-003

> **As** an operator
> **I want** to search products by part of the name and by category, and open one of them
> **so that** I find what to sell or edit without scrolling the whole catalogue

**Acceptance Criteria:**

```gherkin
Scenario 1: Search by part of the name
  Given products whose names contain "tornillo" and others that do not
  When  I search for "tornillo"
  Then  only the products whose name contains that text are listed

Scenario 2: Filter by category
  Given products in several categories
  When  I choose one category
  Then  only the products of that category are listed

Scenario 3: Only active products
  Given a product that was removed (HU-010)
  When  I search
  Then  it is not listed

Scenario 4: Order and paging
  Given more products than fit on one page
  When  I search
  Then  the results are ordered by name, paged, and the total number of matches is returned

Scenario 5: Open one product
  Given a product identifier
  When  I open it
  Then  I see its name, price, stock, category and image key
  And   a removed or unknown product is reported as not found
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-005 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** Q1 is the highest-frequency query that needs a trigram index, planned in T-13 (§6.2); a leading-wildcard search cannot use a B-tree.

---

### HU-007 — Change a product's name and price {#HU-007}

**Epic:** EP-002
**Data model references:** §2.2 (`Product.Rename`, `Product.ChangePrice`), §1 (*Nombre congelado*)

> **As** an administrator **[assumption] S-1**
> **I want** to rename a product or change its price
> **so that** the catalogue stays correct without altering past sales

**Acceptance Criteria:**

```gherkin
Scenario 1: Rename
  Given an active product
  When  I give it a non-empty name
  Then  the name is saved trimmed

Scenario 2: Change the price
  Given an active product
  When  I give it a price greater than zero
  Then  the new price applies to future sales only

Scenario 3: Invalid values
  Given an active product
  When  I give it an empty name or a price of 0 or less
  Then  the change is rejected and the product stays as it was

Scenario 4: Past sales untouched
  Given a sale registered before the change
  When  I open that sale
  Then  its lines still show the old name and the old unit price
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Should Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-005 |
| Affected service(s) | simple-stock-flow-api |

---

### HU-008 — Restock a product {#HU-008}

**Epic:** EP-002
**Data model references:** §2.2 (`Product.Restock`), D-04, ADR-002

> **As** an administrator **[assumption] S-1**
> **I want** to add units to a product's stock
> **so that** there is stock available to sell

**Acceptance Criteria:**

```gherkin
Scenario 1: Add units
  Given a product with 5 units
  When  I restock 20 units
  Then  its stock is 25

Scenario 2: Invalid amount
  Given a product
  When  I restock a quantity that is zero or negative
  Then  it is rejected and the stock does not change
  # a strictly positive quantity is an [assumption]; the model only fixes stock >= 0 after any operation (§2.2)

Scenario 3: Never below zero
  Given any sequence of operations on a product
  When  the stock is read
  Then  it is never below zero, not even for a write made outside the application (ck_product_stock_non_negative, §4)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-005 |
| Affected service(s) | simple-stock-flow-api |

---

### HU-009 — Attach, replace or remove a product image {#HU-009}

**Epic:** EP-002
**Data model references:** D-08, §1 (*Imagen del producto*), §2.2 (`Product.AttachImage`), §7.1, §11 H-2

> **As** an administrator **[assumption] S-1**
> **I want** to give a product an image, replace it or remove it
> **so that** operators recognise the product at a glance

**Acceptance Criteria:**

```gherkin
Scenario 1: Attach
  Given a product without image
  When  I attach an image
  Then  the binary goes to external storage
  And   the product stores only an opaque key, never a path or the bytes

Scenario 2: No image
  Given a product without image
  When  I read it
  Then  its image key is null, never an empty string

Scenario 3: Replace
  Given a product with an image
  When  I replace it
  Then  the old binary is deleted after the change is committed

Scenario 4: Safe order
  Given an image is being removed
  When  the binary cannot be deleted
  Then  the product has no key pointing to it (a dangling key is worse than an orphan binary; §7.1)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Could Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-005 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** storage does not take part in the database transaction, so atomicity is not promised (§7.1). Cleaning up orphan binaries has no owner (H-2).

---

### HU-010 — Remove a product from the catalogue {#HU-010}

**Epic:** EP-002
**Data model references:** ADR-003, §2.2, §5 (FK-3), §7.1, D-03

> **As** an administrator **[assumption] S-1**
> **I want** to take a product out of the catalogue without losing its history
> **so that** it can no longer be sold while past sales and reports stay intact

**Acceptance Criteria:**

```gherkin
Scenario 1: Soft removal
  Given an active product
  When  I remove it
  Then  its row is kept and marked as removed (deleted_at is set)

Scenario 2: Disappears from the catalogue
  Given a removed product
  When  I search or open it
  Then  it is not found (HU-006)

Scenario 3: Cannot be sold
  Given a removed product
  When  a seller tries to sell it
  Then  the sale is rejected (HU-012)

Scenario 4: History intact
  Given a removed product that was sold before
  When  I open an old sale or the report of its period
  Then  the product still appears

Scenario 5: Manual physical delete is blocked
  Given a product that appears in a sale line
  When  someone runs a physical DELETE on it directly in the database
  Then  the DELETE fails (FK-3, RESTRICT)

Scenario 6: Image binary
  Given a removed product that had an image
  When  the removal is committed
  Then  its image binary is deleted (§7.1)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Should Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-005 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** the model contradicts itself on the state of FK-3 and of `deleted_at` (CC-05 in [`../05-architecture/consistency-check.md`](../05-architecture/consistency-check.md)); §13 (2026-09-20) is followed: both are in the engine.

---

### HU-011 — Register a sale {#HU-011}

**Epic:** EP-003
**Data model references:** §2.3, §2.4, §1 (*Venta*, *Línea de venta*), Q3, ADR-002

> **As** a seller
> **I want** to register a sale of one or more products
> **so that** the sale is recorded and the stock is updated in the same step

**Acceptance Criteria:**

```gherkin
Scenario 1: One line
  Given a product with 10 units at a price of 5.00
  When  I register a sale of 3 units of it
  Then  the sale is saved with my username and the current instant
  And   it has one line with quantity 3 and unit price 5.00
  And   the product stock is 7

Scenario 2: Several lines
  Given two different products with enough stock
  When  I register one sale with a line for each
  Then  both stocks are reduced and the sale has two lines

Scenario 3: Total is computed
  Given a saved sale
  When  I read it
  Then  its total equals the sum of quantity times unit price of its lines
  And   no total or subtotal is stored (article VII, §1)

Scenario 4: Same product twice
  Given a sale with two lines for the same product
  When  I register it
  Then  it is rejected and nothing is saved (Sale.AddItem; unique (sale_id, product_id))

Scenario 5: Sale without lines
  Given a sale with no lines
  When  I register it
  Then  it is rejected (Sale.EnsureConfirmable)

Scenario 6: Immutable
  Given a registered sale
  When  I look for an operation to edit or delete it
  Then  none exists (§2.3)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-001, HU-005, HU-008 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** deducting stock and adding the line are **one operation** (§2.3). It touches two aggregates, so a single transaction is an **[assumption]** (see `../05-architecture/overview.md` §7).

---

### HU-012 — Reject a sale that cannot be fulfilled {#HU-012}

**Epic:** EP-003
**Data model references:** §2.2 (`Product.Withdraw`), §2.4, §5 (cardinalities), D-04, ADR-002

> **As** a seller
> **I want** the system to refuse a sale it cannot fulfil, without changing anything
> **so that** stock never goes negative and the books never record goods that were not there

**Acceptance Criteria:**

```gherkin
Scenario 1: Not enough stock
  Given a product with 2 units
  When  I register a sale of 3 units of it
  Then  the whole sale is rejected
  And   no stock changes and nothing is saved

Scenario 2: Quantity not positive
  Given a line with quantity 0 or less
  When  I register the sale
  Then  it is rejected and nothing is saved

Scenario 3: Removed product
  Given a product that was removed
  When  I include it in a sale
  Then  the sale is rejected (§5: the product must not be removed at the time of sale)

Scenario 4: Unknown product
  Given a product identifier that does not exist
  When  I include it in a sale
  Then  the sale is rejected

Scenario 5: Two sellers, one last unit
  Given a product with 1 unit and two sellers selling it at the same moment
  When  both sales are submitted
  Then  at most one succeeds and the stock is never below zero
  # what the losing seller sees (error or retry) is not defined in the model (D-04)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-011 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** concurrency is optimistic with `xmin` as the token; `ck_product_stock_non_negative` is the last barrier (§3, ADR-002). `quantity > 0` is guaranteed by C# only today (T-20).

---

### HU-013 — Keep the sale's values frozen {#HU-013}

**Epic:** EP-003
**Data model references:** §1 (*Nombre congelado*), §2.4, §3 (`sale_item`), ADR-004, D-06

> **As** an administrator
> **I want** each sale line to keep the product name, unit price and category name of the moment of the sale
> **so that** I can rename, reprice or recategorise products without rewriting closed periods

**Acceptance Criteria:**

```gherkin
Scenario 1: Values copied at sale time
  Given a product named "Martillo", priced 12.00, in category "Herramientas"
  When  I sell it
  Then  the line stores name "Martillo", unit price 12.00 and category name "Herramientas"

Scenario 2: Rename afterwards
  Given that sale
  When  the product is renamed to "Martillo pro"
  Then  the old sale line still shows "Martillo"

Scenario 3: Reprice afterwards
  Given that sale
  When  the product price becomes 15.00
  Then  the old sale line still shows 12.00

Scenario 4: Recategorise afterwards
  Given that sale
  When  the product moves to another category
  Then  the old sale line still shows "Herramientas"
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-011 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** `sale_item.category_name` has no foreign key on purpose, or renaming a category would rewrite history. Whether the column already exists is unresolved (CC-03 in the consistency check).

---

### HU-014 — View a sale {#HU-014}

**Epic:** EP-003
**Data model references:** Q6 (§6.1), §2.3, §7

> **As** an operator
> **I want** to open a sale and see its lines
> **so that** I can check what was sold, by whom and when

**Acceptance Criteria:**

```gherkin
Scenario 1: Sale with its lines
  Given a registered sale
  When  I open it
  Then  I see who sold it, when, and every line with product name, quantity and unit price
  And   the total computed from those lines

Scenario 2: Unknown sale
  Given an identifier that does not exist
  When  I open it
  Then  it is reported as not found
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Should Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-011 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** the seller's name is personal data: it appears on receipts but access is restricted (§7). The field travels as `SoldBy` (§3).

---

### HU-015 — List sales in a date range {#HU-015}

**Epic:** EP-003
**Data model references:** Q7 (§6.1), §1 (*Rango de fechas*), §6.2 (`IX_sale_sold_at`)

> **As** an operator
> **I want** to list the sales of a date range
> **so that** I can review what was sold in a period

**Acceptance Criteria:**

```gherkin
Scenario 1: Sales in the range
  Given sales on different dates
  When  I choose a range
  Then  only the sales inside it are listed, newest first

Scenario 2: Paging
  Given more sales than fit on one page
  When  I list them
  Then  they are paged and the total number is returned

Scenario 3: Invalid range
  Given a range whose end is before its start
  When  I list the sales
  Then  it is rejected
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Should Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-011 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** the unpaged variant (Q8) has no consumer and was dropped from the design (§6.1). Whether the range includes its end date is **[assumption]**: not defined.

---

### HU-016 — Sales report by product {#HU-016}

**Epic:** EP-004
**Data model references:** Q9 (§6.1), D-06, ADR-004, DP-02, §1 (*Reporte de ventas*)

> **As** an administrator **[assumption] S-1**
> **I want** to see how much of each product was sold in a date range
> **so that** I know which products move and which bring in the most money

**Acceptance Criteria:**

```gherkin
Scenario 1: Aggregation by product
  Given sales of several products in a range
  When  I ask for the report of that range
  Then  I see, for each product, the units sold and the amount, ordered by amount from highest to lowest
  # the exact columns are an [assumption]; the model says "aggregates by product, ordered by amount descending"

Scenario 2: Not broken down by seller
  Given sales made by different sellers
  When  I read the report
  Then  nothing in it is grouped or filtered by seller (DP-02)

Scenario 3: Nothing sold
  Given a range with no sales
  When  I ask for the report
  Then  the report is empty

Scenario 4: Invalid range
  Given a range whose end is before its start
  When  I ask for the report
  Then  it is rejected

Scenario 5: Computed, not stored
  Given the database schema
  When  I look for a table that holds the report
  Then  none exists: it is computed from the sales when requested (§1, D-06)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Must Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-011 |
| Affected service(s) | simple-stock-flow-api |

> **Note:** Q9 is "the most costly" query (§6.1). It is served by the covering index `sale_item (sale_id, product_id) INCLUDE (quantity, unit_price)` (§6.2).

---

### HU-017 — A closed report never changes {#HU-017}

**Epic:** EP-004
**Data model references:** §11.1 (H-1, closed), ADR-004, D-06

> **As** an administrator
> **I want** the report of a period that already ended to give the same result today and next year
> **so that** I can trust and compare reports across time

**Acceptance Criteria:**

```gherkin
Scenario 1: Rename does not rewrite the report
  Given a report of a closed period that lists "Martillo"
  When  the product is later renamed
  Then  the same report still lists "Martillo"

Scenario 2: Recategorisation inside the range
  Given a product sold in September under "Herramientas" and in October under "Ferretería"
  When  I ask for the report of September to October
  Then  that product appears in two rows, one per frozen category name (§11.1)

Scenario 3: A new sale does not alter what was read
  Given I read the report of a closed range
  When  a sale with a new category label is registered afterwards
  Then  the report of that same closed range is unchanged

Scenario 4: Grouping key
  Given the report query
  When  its grouping is inspected
  Then  it groups by product, product name and frozen category name, with no tie-break rule (§11.1)
```

| Field | Value |
|-------|-------|
| Story Points | To be estimated |
| Priority | Should Have **[assumption]** |
| Target sprint | To be planned |
| Status | Backlog |
| Dependencies | HU-013, HU-016 |
| Affected service(s) | simple-stock-flow-api |

> **Note — open conflict declared by the model (§11.1):** `spec.md` CA-06.1 says "one row per product"; this story produces **more than one row** after a recategorisation. The owner must rewrite CA-06.1 as "one row per product **and frozen label**". Until then, this backlog follows the owner's decision. Known defect **A-7** (product name chosen with the tie-break that H-1 rejected, DP-01) is **reported, not fixed** (§11.1).

---

## Stories deliberately not written

The model closes these questions, so there is no story for them.

| Not written | Why | Cite |
|---|---|---|
| Edit or delete a sale | A sale is an immutable commercial fact | §1 *Venta*, §2.3 |
| Create, rename or delete categories | Fixed set of five, seeded | §2.1, D-10, §4.1 |
| Customer, buyer or buyer data | The sale records the operator, not the buyer | §1 *Usuario*, §7 |
| Prices in several currencies | Single currency by construction | D-05, §3 |
| Report by seller | It would cross personal data | DP-02, §6.3 |
| More product attributes (description, SKU…) | Closed | DP-03 |
| History of catalogue changes (`created_at`, `updated_at`) | No requirement; reopen only if someone asks "who changed this price, and when?" | §8 |
| Grant the `admin` role while the system runs | Decided | DP-04, §11 H-3 |
| Analytics export or anonymisation | Does not exist | §7.1 |

## Open questions for the owner

1. **S-1:** which role may do each operation (catalogue changes, report, sales listing).
2. **HU-017:** rewrite `spec.md` CA-06.1 (§11.1).
3. **HU-012, Scenario 5:** what the losing seller sees on a concurrent last-unit sale.
4. **HU-015:** whether the range includes its end date.
5. **H-2:** who cleans up orphan image binaries (§11).

---

## Correlations

- Non-functional requirements → [`non-functional.md`](non-functional.md)
- Traceability matrix → [`traceability-matrix.md`](traceability-matrix.md)
- Architecture that serves these stories → [`../05-architecture/overview.md`](../05-architecture/overview.md)
- Findings about contradictions in the model → [`../05-architecture/consistency-check.md`](../05-architecture/consistency-check.md)
- Domain entities and rules → [`../02-domain/entities-and-rules.md`](../02-domain/entities-and-rules.md)


