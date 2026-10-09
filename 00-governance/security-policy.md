# Security policy

> Scaffold based on facts in the supplied data model. It does not introduce a JWT, password-strength, encryption, or regulatory requirement absent from the source.

## 1. Security scope
The model covers internal users, admin/seller roles, authentication secrets, and sale attribution (model §§1, 2.3, 2.5, 7). No customer identity is modeled.

## 2. Model-backed controls
- Clear-text passwords never enter the domain; a hash port produces the stored hash (D-09, §2.5).
- password_hash must never appear in logs, responses, projections, or errors, and is never indexed (§7).
- Roles are admin and seller; validation is domain-only today and T-20 is associated with a database check (§2.5, §4).
- The first admin is provisioned at startup from environment credentials, not from a SQL hash literal (§9.2).
- Runtime user creation does not grant admin; the model records 401 without a token and 403 for a seller creating users (§11 H-3 / DP-04, §13 D-3).

## 3. Policies not defined by the model
Authentication protocol, token format, full role-permission matrix, password policy, credential reset/rotation, rate limiting, encryption, incident response, and vulnerability handling: [TODO].

## 4. Owners and review
Security decision owner: [TODO]
Review cadence: [TODO]
Incident contact/process: [TODO]
