# Non-functional requirements

> Reconstructed backwards from [the data model](../spec/data-model.md). Every requirement cites its evidence. **[assumption]** marks a proposed quality requirement or interpretation that the model does not itself establish; it is not a measured service-level objective.

## Requirements

| ID | Requirement | Evidence and limits |
|---|---|---|
| NFR-01 | **Data integrity.** Persisted stock must never be negative, including writes outside the application. Domain validation must reject withdrawals above available stock. | `ck_product_stock_non_negative` and ADR-002, §§2.2, 4. The over-withdrawal rule is domain-only. |
| NFR-02 | **Atomic sale operation.** Registering a sale must keep sale lines and stock movement consistent as one business operation. | `Sale.AddItem`, §2.3. **[assumption]** A database transaction should cover the complete sale; transaction and retry details are not specified. |
| NFR-03 | **Concurrent updates.** Conflicting product updates must be detectable; the system uses PostgreSQL `xmin` as the optimistic-concurrency token. | D-04, ADR-002, §3. **[assumption]** The application should report a conflict and require a fresh read; response format and retry policy are unspecified. |
| NFR-04 | **Authentication secrecy.** Plain passwords must not enter the domain or be stored. Store only a password hash produced through the hash port. | D-09, §§2.5, 7, 9.2. |
| NFR-05 | **Sensitive-data handling.** Never expose `password_hash` in logs, responses, projections, or errors; never index it. Restrict access to usernames, roles, and seller identity. | §7. Authentication secret handling is explicit; access-control implementation details are not. |
| NFR-06 | **Authorization.** User creation must require an authenticated administrator; a seller is forbidden from creating users. Runtime creation of an `admin` role is not allowed. | §13 D-3, §11 H-3 / DP-04. Token format is **[assumption]**; the model only records the observed 401/403 behavior. |
| NFR-07 | **Historical integrity.** Sales and sale lines are immutable and retained indefinitely. Product removal is soft deletion; historical sale values remain frozen. | §1, §§2.2–2.4, §7.1, ADR-003. |
| NFR-08 | **Image consistency.** Persist an opaque image key, not image bytes or a filesystem path. When removing an image, clear and commit the key before deleting the external binary. | D-08, §1, §7.1. Cross-resource atomicity is not promised; orphan-binary cleanup is unresolved (§11 H-2). |
| NFR-09 | **Monetary precision.** Use one currency; round to two decimal places with `AwayFromZero` and store `numeric(18,2)`. Keep domain and database precision aligned. | D-05, §§2.2, 3. |
| NFR-10 | **Time consistency.** Store business timestamps as `timestamptz`; the database server is configured for UTC. | Header and §3. **[assumption]** API timestamps should preserve an unambiguous offset or UTC representation; wire format is not defined. |
| NFR-11 | **Query efficiency.** Implement the access patterns and indexes described by the model, including planned partial/trigram indexes for product search; do not claim a latency target from index definitions alone. | Q1–Q10, §§6.1–6.3. Some indexes and `pg_trgm` are explicitly pending T-13 (§§6.2, 10.3). |
| NFR-12 | **Minimize stored data.** Do not add customer identity, seller-report breakdown, currency columns, product attributes beyond the decided set, or `created_at`/`updated_at` without a new decision. | D-05, DP-02/03, §§1, 8, 12. |
| NFR-13 | **Availability, backup, recovery, capacity, and numeric performance targets.** No objectives can be stated from the model. | **[assumption]** These are open operational requirements; the data model defines no SLO, RPO/RTO, workload size, or latency threshold. |

## Verification notes

- Verify database-enforced constraints against §4 and the live-engine measurements in §10; the model records dated outputs and internal inconsistencies (consistency-check CC-03–CC-07).
- Verify that a sale preserves the stock/line invariant under a failed write and concurrent update (**[assumption]** test scenarios; expected business behavior follows §2.3 and D-04).
- Verify secret redaction and access restrictions against §7; the model does not specify a logging framework or HTTP error schema.
- Treat T-20, T-12, and T-13 as planned work, not as quality already delivered (§§4–6).