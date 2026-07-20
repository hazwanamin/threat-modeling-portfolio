\# SaaS API - element inventory



\## External entities

\- Tenant A (authenticated user)

\- Tenant B (authenticated user)



\## Processes

\- SaaS API (validates and processes requests, resolves tenant context)



\## Data stores

\- Tenant data DB (shared, tenant-scoped by tenant\_id column)



\## Data flows

1\. Tenant A -> SaaS API : authenticated request for own data

2\. SaaS API -> Tenant data DB : reads/writes Tenant A's records, scoped by tenant\_id

3\. Tenant B -> SaaS API : authenticated request for own data

4\. SaaS API -> Tenant data DB : reads/writes Tenant B's records, scoped by tenant\_id

5\. Tenant A -> SaaS API : modifies object ID in request to target Tenant B's record (abuse case)

6\. SaaS API -> Tenant data DB : query executes without tenant\_id filter, returns Tenant B's data (abuse case)



\## Trust boundaries

\- Single trust boundary drawn around internal infrastructure

\- Both tenants treated as external/untrusted, sitting outside this boundary

