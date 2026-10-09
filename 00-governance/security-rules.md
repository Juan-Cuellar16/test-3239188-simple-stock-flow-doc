# Security rules

> Model-backed handling requirements. Items marked [assumption] need owner confirmation before becoming policy.

## 1. Credentials and password hashes
- Never pass clear-text passwords into the domain; use the hash port (D-09, §2.5).
- Never expose password_hash in logs, responses, projections, or errors; never index it (§7).
- Do not version initial credentials or a precomputed hash. Provision the first admin from environment credentials at startup (§9.2).
- [assumption] Do not print environment credentials in startup diagnostics; logging behavior is not specified.

## 2. Identity and authorization
- Accept only admin and seller roles; current validation is domain-only (model §§2.5, 4).
- Do not grant admin during ordinary runtime user creation (DP-04, §11 H-3).
- Preserve the model's observed outcomes: unauthenticated user creation returns 401; seller user creation returns 403 (§13 D-3).
- Do not claim the database enforces lowercase username or role membership until the T-20 state is verified (§2.5, §4, CC-07).
- [assumption] Enforce authorization at the server-side application boundary; the model says authorization lives in the API but does not define endpoint policies (§9.2).

## 3. Personal and confidential data
- Treat username and sale attribution as personal data; role is internal-confidential (§7).
- Do not add customer identity or seller-level report data (DP-02, §§1, 7).
- [assumption] Review future logs and event payloads for personal data; detailed procedures are [TODO].

## 4. Validation and database integrity
- Keep engine, domain-only, and pending rules distinct (model §§2, 4).
- Preserve the database check that prevents negative stock (§2.2, §4, ADR-002).
- Check current schema state against the consistency findings before claiming disputed constraints or indexes are present (05-architecture/consistency-check.md).

## 5. Additional controls to decide
Authentication protocol, password reset, encryption, CORS, rate limiting, lockout, security logging, and incident response: [TODO; not specified in the data model].
