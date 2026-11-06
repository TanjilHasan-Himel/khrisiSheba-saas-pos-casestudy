# 06 — Security, Privacy and Compliance

## Threat model

Cross-tenant data leaks; horizontal privilege escalation; fraudulent stock adjustments; replayed payments/sales; leaked staff credentials; malicious file uploads; incorrect customer ledger; unauthorized export; insecure POS integration credentials; lost devices.

## Required controls

1. Tenant context derived from authenticated active membership and validated store access, never raw request header alone.
2. RLS on every tenant-owned table, plus service authorization and composite foreign keys.
3. Password/session security, MFA for platform operators and ideally owners; secure cookies, CSRF protection where relevant, rate limiting and lockout policy.
4. Principle of least privilege with permission matrix for read/create/post/reverse/export/manage.
5. Append-only audit of posted sales, inventory, returns, dues, role changes and support access.
6. Idempotent financial writes; row locking; server-side total calculation; signed external webhook verification when implemented.
7. Encrypt backups and secrets; no API tokens in browser/local storage; storage object paths and signed URLs scoped to tenant.
8. PII minimization, data retention/deletion policies, privacy notices, export controls, incident response and breach procedures.
9. Automated isolation tests using at least two tenants and multiple stores; test direct object reference attacks and background jobs.
10. Restore drills and documented backup/rollback runbooks before real-money launch.

## Agricultural retail regulatory checklist (verification required)

Check applicable Bangladesh licensing, registration, product labeling, restricted pesticide sale, storage, recordkeeping, fertilizer/seed rules, invoicing/tax, data/privacy and payment compliance with qualified local professionals and authorities. Do **not** infer legality from product catalog availability or implement agricultural advice as regulatory assurance.

## Permission matrix (baseline)

| Action                     | Owner | Manager            | Sales  | Stock | Platform operator |
| -------------------------- | ----- | ------------------ | ------ | ----- | ----------------- |
| View tenant reports        | Yes   | Scoped             | No     | No    | No by default     |
| Manage staff               | Yes   | Optional delegated | No     | No    | No                |
| Receive purchases          | Yes   | Yes                | No     | Yes   | No                |
| Post manual sales          | Yes   | Yes                | Yes    | No    | No                |
| Collect dues               | Yes   | Yes                | Scoped | No    | No                |
| Approve adjustment/return  | Yes   | Scoped             | No     | No    | No                |
| Manage tenant subscription | Yes   | No                 | No     | No    | Metadata only     |

Permissions must be enforced in API services, not hidden buttons only.
