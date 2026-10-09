# Security policy

> **Traceability:** This policy records the security boundaries and controls stated in the supplied [data model](../spec/data-model.md). It does not claim compliance with an external security standard. **[assumption]** marks recommendations that the model does not define.

## 1. Purpose and scope

This policy describes how Simple Stock Flow handles the security concerns visible in its data model: internal user identity, the `admin` and `seller` roles, password hashes, sale attribution, personal data, and product-image references (§1, §§2.3, 2.5, §7).

The system does not model customer or buyer accounts. A sale identifies the internal operator who recorded it (§1, §7). Security controls or integrations beyond this boundary require a separate approved decision.

## 2. Security principles

### 2.1 Protect authentication secrets

The domain never receives a clear-text password. A hash port produces and verifies the stored password hash (D-09, §2.5). The `password_hash` must never appear in logs, responses, projections, or error messages, and must never be indexed (§7).

### 2.2 Limit access by role

The modeled role set is `admin` and `seller` (§1, §2.5). User creation without authentication returns `401`; a seller attempting to create a user receives `403` (§13 D-3). The model does not define a complete permission matrix for catalogue, sales, and report operations; that remains an owner decision (backlog assumption S-1).

### 2.3 Do not grant admin at runtime

The initial administrator is provisioned at application startup from environment credentials. A precomputed hash is not seeded in SQL, and the `admin` role is not granted through ordinary runtime user creation (§9.2, §11 H-3 / DP-04).

### 2.4 Minimize personal information

`username` and sale attribution are classified as personal data. `role` is internal-confidential. The password hash is an authentication secret (§7). The report is not broken down by seller, and customer identity is not part of the model (DP-02, §1, §7).

### 2.5 Preserve business-data integrity

Stock must not become negative; the database check `ck_product_stock_non_negative` is the final barrier for that invariant (§2.2, §4, ADR-002). Sales are immutable and retained indefinitely. Products are removed through soft deletion rather than physical deletion (§2.2–2.3, §7.1, ADR-003).

## 3. Identity and account handling

- Accept only the roles `admin` and `seller` (§2.5).
- Normalize usernames to lowercase and trim surrounding spaces through domain behavior. The model classifies this rule as domain-only and associates an engine check with T-20 (§2.5, §4).
- Enforce username uniqueness through the unique index described in the model (§2.5, §4).
- Create the initial administrator during startup using credentials from the environment; do not store those credentials or a precomputed hash in the repository (§9.2).
- Create seller accounts through an administrator; do not grant `admin` during ordinary runtime user creation (§11 H-3 / DP-04).
- Preserve the modeled authorization outcomes for user creation: no token returns `401`, and a seller receives `403` (§13 D-3).

**[Assumption]** Environment credentials should not be written to startup logs or diagnostic output. The model specifies their source but does not define logging behavior (§9.2).

## 4. Data protection and retention

| Data | Model classification or handling |
|---|---|
| `password_hash` | Authentication secret. Never log, return, project, or include in errors; never index. Its legitimate use is verification through the hash port (§7). |
| `username` | Personal data identifying an internal operator; access is restricted (§7). |
| `sold_by` / future `sold_by_username` | Personal data identifying the operator on a sale; access is restricted. Sale attribution is retained indefinitely (§3, §7). |
| `role` | Internal-confidential because it reveals privilege level (§7). |
| Sale and sale-line data | Commercial data retained indefinitely; sales and lines are not edited or deleted (§2.3–2.4, §7.1). |
| Product data | Not classified as sensitive in the model; product removal is soft deletion, not physical deletion (§7, §7.1). |
| Product image | Binary is stored externally; the database contains only an opaque key. Clear and commit the key before deleting the binary (§1, §7.1, D-08). |

The image store does not participate in the database transaction. A failed binary deletion may leave an orphan image, and no cleanup process is defined (§7.1, §11 H-2).

## 5. Rule enforcement and verification

The model distinguishes rules enforced by the database, rules enforced only by the domain, and pending work (§§2, 4). A domain-only rule can be bypassed by a direct database write; do not present it as an engine guarantee.

The model classifies positive product price, positive sale quantity, non-empty category name, role membership, and lowercase username as domain-only, with T-20 associated with adding engine checks (§2.1–2.5, §4). The current implementation status of some schema items is disputed in CC-05–CC-07; verify the model’s measurement section before claiming those controls are present (§10).

## 6. Security controls not specified

The model does not define:

- Authentication protocol, token format, or token/session lifetime.
- Password length, complexity, expiration, reset, or rotation rules.
- Encryption algorithms or key-management procedures.
- Rate limiting, account lockout, CORS, or public-endpoint policy.
- Detailed permissions for each operation and role.
- Security logging, vulnerability reporting, incident response, or breach notification.
- A cleanup and retry process for orphan image binaries.

Do not copy values for these controls from another project's policy and present them as Simple Stock Flow requirements. Record an approved decision before treating one as a project rule (§7, §12).

## 7. Responsibilities and open decisions

| Topic | Status |
|---|---|
| Owner for security-policy decisions | **[TODO: identify]** |
| Full `admin` / `seller` permission matrix | **[TODO: owner decision; see S-1]** |
| Authentication mechanism and token/session design | **[TODO: not defined by the data model]** |
| Password reset and credential rotation | **[TODO: not defined]** |
| Logging and incident-response procedure | **[TODO: team policy required]** |
| Orphan-image cleanup owner | **[TODO: owner decision; see §11 H-2]** |

**[Assumption]** Review this policy whenever identity, authorization, or personal-data handling changes. Update the affected requirements and architecture documents at the same time so that the security description remains consistent.