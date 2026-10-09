# Architecture closure - cross-check

> Traceability: cite the data model on every statement (section 2.3, FK-2, D-04). Mark anything that is not in the model as **[assumption]**.

> **Purpose:** close the loop of the challenge (step 6). Check that [`overview.md`](overview.md) is consistent
> with (a) the data model, (b) the governance template, and (c) every document written before it (`01-context` … `04-requirements`).
> **Source of truth:** [`../spec/data-model.md`](../spec/data-model.md). Where the model contradicts itself, the model's own rule applies:
> **the engine wins** (header, §10) — but this repository cannot query the engine, so every conflict below gets a
> **resolution adopted** (our working reading), explicitly marked as an **[assumption]** until measured with the queries of §10.
> **How to cite:** `§x` = section of the data model · `T-xx` = task named by the model · `CC-xx` = finding in this document.

**Status of this document:** parts 1–4 are complete. **Part 5 (cross-document check) is open** until
`01-context`, `02-domain`, `03-product` and `04-requirements` are written; its checklist is ready.

---

## 1. Summary

| Check | Result |
|---|---|
| Does `overview.md` say anything the model does not support? | **No.** Every claim is cited or marked **[assumption]** (part 3) |
| Is the data model consistent with itself? | **No.** 10 findings (part 2): 3 high, 5 medium, 2 low |
| Does `overview.md` follow the governance template? | **Yes**, with 3 justified deviations (part 4) |
| Are the other documents consistent with `overview.md`? | **Pending** (part 5) |

**Why the model contradicts itself.** §13 (2026-09-20) records that the marks of three families of debt were corrected
**in place**, on 2026-09-20 — but the rest of the text, and the verification output of §10 (2026-09-19), were not updated.
Most findings are this one cause seen from different places.

---

## 2. Findings inside the data model

Severity: **High** = an implementer would build the wrong thing · **Medium** = a reader would draw a wrong conclusion · **Low** = cosmetic or traceability.

| ID | Where | What the model says (A) | What it also says (B) | Sev. | Resolution adopted |
|---|---|---|---|---|---|
| **CC-01** | §3 title, §12 · §3 `xmin` row, §3.2, §10.1 · §8 link | "**22** columns" | "does not count among the **21**"; the query "returns **21**"; the link anchor in §8 says `las-21-columnas` and does not match the heading | Low | 21 were **measured** on 2026-09-19 (§10.1). 22 reconciles as 21 + `product.deleted_at` (T-09) **if** `sale_item.category_name` is still pending (see CC-03). Never quote a total without its date |
| **CC-02** | §3, table `product`, row `category_name` | A `product.category_name` column: "frozen label at the moment of sale… no FK on purpose… renaming the category would rewrite history" | §1 and DP-03: a product has "name, price, stock, category and image, **and nothing else**"; §10.1 shows 6 `product` columns, none of them `category_name`; §5 and §2.4 place `category_name` on `sale_item` | **High** | **Copy-paste error.** `category_name` belongs **only** to `sale_item`. `product` gets no such column. `overview.md` already assumes this |
| **CC-03** | §2.4 · §3 `sale_item` row | `sale_item.category_name`: "**engine** (T-11)" | Same column: "**pending (T-11)**… `NOT NULL` is free today because the table is empty" | Medium | **Unknown.** Treated as _pending_ (it is not among the items §13 declares settled). Measure with §10.1 |
| **CC-04** | §6.2, §10.3, §12 · §2.3, §4, §13 D-2 | The covering unique index `sale_item (sale_id, product_id) INCLUDE (quantity, unit_price)` "**is missing (T-13)**"; "three indexes are missing, not five" | The same index "exists since **T-20**, with `INCLUDE (quantity, unit_price)`" | Medium | Treated as **existing** (§13 D-2 is the later record). Consequence: only the two `product` indexes are certain to be missing. Whether `IX_sale_item_sale_id` was dropped in the same migration is **not stated** |
| **CC-05** | §3 `sale_item` · §4 intro and rows · §5 intro and FK-3 text · §6.2 · §6.3 · §7.1 | Marks say **engine**: `deleted_at` (T-09), `sale_id NOT NULL`, the unique index, `FK-3` (T-20) | The prose beside them still says the opposite: `sale_id` "nullable — a defect"; `product_id` "no FK today"; "eight constraints… two FKs"; "four FKs planned, **two exist**"; FK-3 "was never implemented"; products' soft delete "pending T-09" (§6.2, §6.3, §7.1) | Medium | **§13 wins** (it is the latest record, 2026-09-20). Read the marks, ignore the stale prose |
| **CC-06** | §10 (all outputs) · header date | Outputs dated **2026-09-19**: 21 columns, **8 constraints**, 12 indexes, `sale_id` nullable. §10 says "more or fewer than eight rows → the document is broken" | §13 (2026-09-20) says the engine already has more: `deleted_at`, `NOT NULL`, the composite index, `FK-3` | **High** | The verification recipe of §10 **raises a false alarm today**. Expected now (**[assumption]**): ≥ 9 constraints (5 PK, 3 FK, 1 CHECK), more columns and indexes. Re-run the queries and update the counts before trusting them |
| **CC-07** | §2.2, §2.1, §2.5, §4 (T-20) · §13 D-2 | Five rules are "domain only · **T-20**": `price > 0`, `quantity > 0`, non-empty `category.name`, `role` in the closed set, lower-case `username`. "They are five `CHECK`s and one index" | §13 D-2 says three **other** T-20 items are done (`NOT NULL`, composite index, `FK-3`). **Nothing says whether the five `CHECK`s were done** | **High** | Treated as **not done** (the model keeps classifying them "domain only"). `overview.md` §4 and AT-001 carry this as an **[assumption]** |
| **CC-08** | §9.2 · §11 H-3 / §13 D-3 | §9.2: authorisation "lives in the API and **is broken today**: user creation is anonymous (defect A-1)" | §13 D-3: "A-1 is **closed**: no token → 401, `seller` → 403"; H-3 closed by DP-04 | Medium | **A-1 is closed.** The sentence in §9.2 is stale. Security statements in our docs follow §13 |
| **CC-09** | Whole model | Cites D-01…D-10, DP-01…DP-04, A-1, A-7, T-xx, constitution articles (VII, IX, X, XI) | Their definitions live in `plan.md`, `tasks.md`, `constitution.md`, `spec.md`, `HANDOFF-TECNICO.md` — **not delivered** | Low | Only what the model says about each is used. **D-01, D-02 and the text of DP-01 cannot be recovered**; `overview.md` §10 says so |
| **CC-10** | §11.1 | Owner's decision: the report groups by the **frozen** `category_name`; a recategorised product gives **more than one row** | `spec.md` CA-06.1 says "**one row per product**" — the model itself says the two signed statements contradict each other and "the owner must decide" | Medium | **Declared by the model, not resolved.** `04-requirements` adopts the owner's decision and flags the open rewrite. See part 5 |

---

## 3. Claim-by-claim check of `overview.md` against the model

**✔** supported by the model · **A** marked **[assumption]** in the overview (inference, not contradiction) · **⚠** touched by a finding above.

| # | Claim in `overview.md` | Model evidence | Verdict |
|---|---|---|---|
| 1 | Hexagonal style | §2.5, §0, §2.1, §2.3, D-06, §12 (never stated in words) | ✔ inferred · **A** |
| 2 | One service, one database, no broker/gateway | One DB and one schema (§10, §12); nothing else mentioned | ✔ by absence · **A** |
| 3 | C#, EF, PostgreSQL 16 | §0, §2.4, §3.1, header | ✔ |
| 4 | HTTP API with token authentication | §9.2, §12, §13 D-3 show its effect only | **A** |
| 5 | Actors: `admin`, `seller`; no customer | §1 *Rol*, *Usuario*; §7 | ✔ |
| 6 | Four aggregates/entities and their boundaries | §2.1–§2.5, §5 | ✔ |
| 7 | `Category` is not an aggregate | §2.1 | ✔ |
| 8 | Ports and their capabilities | Q1–Q10 (§6.1), D-06, D-08, D-09 | ✔ capability · names **A** |
| 9 | Q8 dropped | §6.1 | ✔ |
| 10 | Rule layers (engine / domain / API) | §4, §2.2, §2.3, §9.2 | ✔ · T-20 state ⚠ **CC-07** |
| 11 | Principles P1–P8 | Header, §3, §3.2, §1, §2, §5, §7, §8 | ✔ |
| 12 | Optimistic concurrency with `xmin` | §3, §6.1, header (ADR-002) | ✔ |
| 13 | Soft delete with `deleted_at` | §2.2, §3, ADR-003 | ✔ (read the marks — **CC-05**) |
| 14 | Frozen name, price and category in `sale_item` | §1, §2.4 | ✔ · `category_name` state ⚠ **CC-03** |
| 15 | Report: `GROUP BY product_id, product_name, category_name` | §11.1 | ✔ · ⚠ **CC-10** |
| 16 | Image deletion order; no atomicity | §7.1 | ✔ |
| 17 | `Money` rounding; single currency | §2.2, D-05 | ✔ |
| 18 | `pg_trgm` installed in the same migration | §6.2 | ✔ |
| 19 | First admin created at startup from the environment | §9.2, D-10 | ✔ |
| 20 | Debt AT-001 … AT-010 | Each row cites its section | ✔ · AT-003 and AT-005 ⚠ **CC-04**, **CC-03** |
| 21 | Repositories `simple-stock-flow-api`, `-infra` | §10, §12 | ✔ |
| 22 | Logging, tracing, health checks, error format "not defined" | The model is silent | ✔ (honest gap) |

**Correction applied to `overview.md` because of this check:** AT-003 listed the covering unique index as missing, while §4 of the
same overview said it already exists. AT-003 now states the conflict and points to **CC-04**.

---

## 4. `overview.md` against the governance template

| Point | Template | `overview.md` | Verdict |
|---|---|---|---|
| Section layout (9 sections + correlations) | `05-architecture/overview.md` | All present, same order | ✔ |
| Language | English for all documentation (`documentation-rules.md` §2, template ADR-001) | English | ✔ |
| "Do not state what is not defined" (`documentation-rules.md` §4, §8) | Rule | Gaps are written as "not defined"; inferences are marked **[assumption]** | ✔ |
| Folder layout of the hexagon | `infrastructure/adapters/out/…` | `src/adapters/outbound/persistence/` (§12) | **Deviation, justified:** the model reports the real paths; the model wins |
| ADR numbering | Template ADR-001 = *documentation language* | The model's ADR-001 = *schema ownership* | **Name clash.** Our ADRs (named by the model, not delivered) are cited as "the model's ADR-00x". If ADRs are written here, renumber or prefix them |
| Style ADR (`ADR-001-architectural-style.md`) | Referenced by the template | Does not exist | **Gap, recorded as AT-009** |
| Template principles *API-first*, *Database per service*, *Observability* | Listed | Not adopted, with the reason | **Deviation, justified** (single service; contract and observability not delivered) |

---

## 5. Cross-document check — **OPEN** (complete after `01` … `04` exist)

Run this checklist once the other documents are written. Mark each row ✔ / ✖ and fix the **later** document, not the model.

| # | Check | Compare | Status |
|---|---|---|---|
| X-1 | Every aggregate/entity in `02-domain` appears in `overview.md` §3 with the same name and boundary | `02-domain/entities-and-rules.md` ↔ overview §3 | Pending |
| X-2 | Every rule in `02-domain` is placed in a layer (engine / domain only / pending) exactly as in overview §4 | `02-domain` ↔ overview §4 | Pending |
| X-3 | Every domain event in `02-domain` is either supported by a model fact or marked **[assumption]**; overview §3 says **no asynchronous messaging** | `02-domain/domain-events.md` ↔ overview §3 | Pending |
| X-4 | Every user story maps to a port or an access pattern (Q1–Q10) in overview §4 | `04-requirements/user-stories.md` ↔ overview §4 | Pending |
| X-5 | Every NFR is backed by an architectural decision or principle (P1–P8, §7 cross-cutting) | `04-requirements/non-functional.md` ↔ overview §5, §7 | Pending |
| X-6 | Report stories say "one row per product **and frozen label**" (CC-10), not "one row per product" | `04-requirements` ↔ model §11.1 | Pending |
| X-7 | Security statements say A-1 is **closed** (CC-08), `admin` is not granted at runtime (DP-04) | `04-requirements`, `01-context` ↔ model §13 | Pending |
| X-8 | The product vision does not promise what the scope excludes (multi-currency, seller breakdown, audit columns, customer data) | `03-product/vision.md` ↔ `01-context/scope.md` ↔ overview §9 | Pending |
| X-9 | Glossary terms use the model's canonical names (`sale`, `sale_item`, `product`, `category`, `user`) | `01-context/glossary.md` ↔ model §0, §1 | Pending |
| X-10 | Actors in `01-context` = actors in overview §2 (`admin`, `seller`; no customer) | `01-context` ↔ overview §2 | Pending |
| X-11 | Nothing in `01-context` … `04-requirements` states a technology the overview marks **[assumption]** as if it were a fact (HTTP, token) | All ↔ overview §1, §7 | Pending |
| X-12 | Every `CC-xx` finding is reflected in the document that depends on it | This file ↔ all | Pending |

---

## 6. Questions for the owner

1. **CC-02:** confirm `category_name` exists only on `sale_item` (and remove it from the `product` table of §3).
2. **CC-03, CC-04, CC-07:** run the queries of §10 and report the real state: `sale_item.category_name`, the composite index, and the five `CHECK`s.
3. **CC-10:** rewrite `spec.md` CA-06.1 to "one row per product **and frozen label**".
4. **CC-06:** refresh the outputs and counts of §10 (and the header date) after the `CC-05` clean-up.

## 7. How to re-run the measurement

The queries and the way to read them are in the model's §10 (columns, constraints, indexes and extensions, row counts).
Expected values **today** are an **[assumption]** derived above (CC-06); the real output replaces them.
If the real output differs from the model's marks, **the document is broken and the engine wins** (header, §10).
