\# Payment API: Threat Model



\## System overview

Client app -> Payment API -> Payment DB / CyberArk vault -> external Payment gateway.

See diagram: payment-api-dfd.png



\## Scope

\- In scope: Payment API process, Payment DB, CyberArk vault, data flows to/from User and Payment gateway

\- Out of scope: internal network hardening, physical security, Payment gateway's own infrastructure



\## Risk rating method: DREAD

Each threat scored 1-3 (Low/Med/High) on: Damage, Reproducibility, Exploitability, Affected users, Discoverability.

Overall rating = average, rounded.



\## Threats



| # | Element | STRIDE | Threat | DREAD (D/R/E/A/D) | Rating | Mitigation | Detection (Splunk) |

\------------------------------------------------------------------------------------------------

| 1 | User -> API | S | Session hijacking via stolen JWT/session token | 3/2/2/2/2 | Med | Short-lived tokens, secure cookie flags | Same session token used from divergent IP/geo in short window |

| 2 | User -> API | T | Request parameter tampering — transaction amount, account ID modified client-side | 3/3/2/3/2 | High | Server-side validation, signed requests | Mismatch between client-submitted amount and server-recomputed amount |

| 3 | User -> API | I | Verbose error messages leak stack trace or DB structure | 2/3/3/2/3 | High | Generic error responses, no verbose output to client | Error response bodies matching stack-trace patterns |

| 4 | User -> API | D | Checkout endpoint flooded | 2/2/3/3/3 | High | Rate limiting, WAF | Spike in requests per IP/session on checkout endpoint |

| 5 | Payment API (process) | R | Refund or price override with no attribution to actor | 3/1/2/1/3 | Med | Structured logging with request ID, commit hash, actor bound to transaction | Privileged action logged with missing/null actor field |

| 6 | Payment API (process) | E | Process runs with excess privilege — compromise gives lateral reach | 3/1/1/1/1 | Low-Med\* | Least-privilege service account scoped to payment processing only | CyberArk vault access from this process outside expected scope |

| 7 | API -> Payment DB | T | Direct DB write bypassing app-layer validation | 3/1/1/2/2 | Med | Parameterized queries, least-privilege DB user | DB writes outside expected app service account pattern |

| 8 | API -> Payment DB | I | Card data/PII exposure via misconfigured backup or overbroad query | 3/1/1/3/2 | Med | Encryption at rest, restrictive access controls | Bulk export or unusually large query against payment DB |

| 9 | API -> CyberArk vault | S | Compromised service account fetches vault credentials impersonating the API | 3/1/1/1/1 | Low-Med\* | mTLS/cert-based service identity, credential rotation | Vault access from unexpected source IP or process hash |

| 10 | API -> CyberArk vault | E | Lateral movement from app tier to vault grants broader access than intended | 3/1/1/1/1 | Low-Med\* | Network segmentation, scoped vault access policies | CyberArk PAM events correlated against ServiceNow change records for unauthorized privileged access |

| 11 | API <-> Payment gateway | S | API or gateway identity spoofed at the boundary | 3/1/2/2/2 | Med | mTLS or signed API keys, IP allowlisting | Gateway calls without valid mTLS cert chain |

| 12 | API <-> Payment gateway | T | MITM alters transaction amount in transit | 3/1/1/2/1 | Med | TLS 1.2+, certificate pinning | Reconciliation mismatch between sent and gateway-confirmed amount |

