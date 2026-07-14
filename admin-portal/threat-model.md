\# Admin Portal — Threat Model



\## System overview

Admin user / Regular user -> Admin portal backend -> Authorization/RBAC check

\-> Roles \& Permissions DB. Backend also writes to Audit log store on every

privileged action.

See diagram: admin-portal-dfd.png



\## Scope

\- In scope: Admin portal backend process, Authorization/RBAC check process,

&#x20; Roles \& Permissions DB, Audit log store, data flows to/from Admin user and

&#x20; Regular user (including the abuse-case flow)

\- Out of scope: underlying network segmentation, physical security, identity

&#x20; provider's own infrastructure (assumes SSO/IdP integrity as a dependency,

&#x20; not analyzed here)



\## Risk rating method: DREAD

Each threat scored 1-3 (Low/Med/High) on: Damage, Reproducibility,

Exploitability, Affected users, Discoverability.

Overall rating = average, rounded — overridden to reflect real risk where the

mechanical average understates it (e.g. high Damage masked by low

Reproducibility/Discoverability due to requiring prior foothold or insider

access). Overrides marked with \*.



\## Threats



| # | Element | STRIDE | Threat | DREAD (D/R/E/A/D) | Rating | Mitigation | Detection (Splunk) |

\------------------------------------------------------------------------------------------------

| 1 | Regular user -> Admin portal backend (abuse case) | E | Regular user calls a privileged admin endpoint directly with a valid but non-admin session; if the backend trusts a client-supplied role claim instead of re-verifying server-side, the request succeeds (BFLA) | 3/3/2/3/2 | High | Enforce authorization server-side on every privileged endpoint via the RBAC check — deny by default, verify role from Roles \& Permissions DB on each request | Privileged action logged with an actor whose role in the DB doesn't match the required role for that action |

| 2 | Admin portal backend (process) | R | Privileged actions occur with no reliable binding between the action and the identity that performed it | 2/2/2/2/1 | Med | Every privileged action logged with actor resolved from authenticated session; admin actions gated through PIM/PAM (CyberArk) with recorded sessions | auditd/Splunk use case correlating high-privilege actions to PIM session ID to positively identify the actor |

| 3 | Admin portal backend -> Audit log store | T | Admin with backend access alters or deletes log entries after the fact, defeating the log and enabling undetected repudiation | 3/1/2/2/1 | High\* | Write logs to an append-only/immutable store the backend itself cannot modify or delete | Alert on any modification/deletion attempt against the log store, and on sequence-number gaps in the log stream |

| 4 | Authorization/RBAC check (process) | E | Check contains a logic flaw (fails open on error, or validates role as an unvalidated string) allowing a crafted request to bypass the role check | 3/1/2/2/1 | Med | Fail closed on any error in the check; explicitly test for fail-open conditions | Elevated error rate on Authorization check followed by a subsequent successful privileged action |

| 5 | Authorization check -> Roles \& Permissions DB | T | Direct write access to the roles table (compromised DB creds, insider access) lets an attacker grant themselves admin role, bypassing the application layer entirely | 3/1/1/2/1 | High\* | Least-privilege DB service account (read-only for the check where possible); restrict direct write access to a controlled migration path only | Direct write to roles table outside the expected app service account / ORM pattern |

| 6 | Admin user -> Admin portal backend | S | Admin credentials or session token compromised (phishing, malware, token theft); attacker impersonates a legitimate admin | 3/2/2/2/2 | Med-High | MFA enforced on all admin accounts, short-lived session tokens, step-up auth for high-risk actions | Admin session activity from new/unrecognized device or divergent geo/IP in a short window |

