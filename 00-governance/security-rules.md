# Security rules

> **Traceability:** These rules are based on the supplied [data model](../spec/data-model.md). **[assumption]** marks a recommended control that the model does not explicitly define. The data model is inconsistent about some implementation status; see CC-05 and CC-07 in `05-architecture/consistency-check.md`.

## 1. Passwords and credentials

### SR-01 — Keep clear-text passwords out of the domain

The domain must never receive a clear-text password. Password hashing and verification must go through the hash port (D-09, §2.5).

### SR-02 — Protect the stored password hash

Never include `password_hash` in logs, API responses, projections, or error messages. Never create an index on `password_hash`. Its only legitimate use is password verification through the hash port (§7).

### SR-03 — Provision the initial administrator safely

Create the initial administrator during application startup using credentials supplied through the environment. Do not seed a precomputed password hash in SQL or commit credentials to the repository (§9.2, D-09, D-10).

**[assumption]** Startup diagnostics must not print environment credentials. The model specifies where the credentials come from, but does not define logging behavior (§9.2).

## 2. Roles and authorization

### SR-04 — Accept only the modeled roles

The valid role set is `admin` and `seller`. Role validation currently lives in domain code; the model associates moving this rule to the database with T-20 (§1, §2.5, §4).

### SR-05 — Do not grant administrator role at runtime

Ordinary user creation must not grant the `admin` role. The initial administrator is provisioned by deployment; administrators create sellers (DP-04, §9.2, §11 H-3).

### SR-06 — Preserve the modeled user-creation authorization behavior

User creation without authentication must return `401`. A seller attempting to create a user must receive `403` (§13 D-3).

**[assumption]** Enforce authorization at the server-side application boundary. The model says authorization lives in the API, but does not define routes or the complete permission matrix (§9.2; backlog assumption S-1).

### SR-07 — Do not overstate username enforcement

Usernames are normalized to lowercase and trimmed by domain behavior. The model classifies this as domain-only and tracks an engine check in T-20; do not claim the database enforces it until the current state is verified (§2.5, §4, CC-07).

## 3. Personal and confidential information

### SR-08 — Restrict access to personal and internal data

Treat `username` and sale attribution (`sold_by`, later `sold_by_username`) as personal data. Treat `role` as internal-confidential (§7).

### SR-09 — Keep customer data and seller reporting out of scope

Do not add customer or buyer identity to sales. Do not expose a seller-level breakdown in the sales report; the model explicitly excludes that breakdown (DP-02, §1, §7).

**[assumption]** Review future logs, exports, and event payloads for personal information before enabling them. The model does not define a logging or export policy (§7, §12).

## 4. Data integrity and validation

### SR-10 — Preserve the distinction between rule layers

For each invariant, document whether it is enforced by the database, only by the domain, or is pending. Do not describe domain-only or pending rules as database guarantees (model header, §§2, 4).

### SR-11 — Preserve the nonnegative-stock constraint

The database must retain the `ck_product_stock_non_negative` check as the last barrier against negative stock (§2.2, §4, ADR-002).

### SR-12 — Verify disputed schema state before relying on it

The model contains stale or contradictory descriptions of soft deletion, the sale-item product foreign key, and other schema state. Check the consistency findings and the model’s measurement procedure before claiming a disputed control is present (CC-05–CC-07; model §10).

## 5. Security controls not defined by the source

The data model does not specify an authentication protocol or token format, password complexity rules, credential reset or rotation, encryption algorithms, rate limiting, account lockout, CORS, or an incident-response process (§7, §12).

Do not treat values copied from another project as Simple Stock Flow requirements. Record a future decision with its approved source; until then, mark a proposed control as **[assumption]**.

## 6. Open decisions

- Authentication mechanism and token/session format: **[TODO]**.
- Complete permission matrix for `admin` and `seller`: **[TODO; see S-1]**.
- Password reset and credential rotation process: **[TODO]**.
- Logging, vulnerability reporting, and incident response: **[TODO]**.