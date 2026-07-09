\# Admin Portal - element inventory



\## External entities

\- Admin user (legitimate)

\- Regular user (low-privilege, attempting escalation - abuse case)



\## Processes

\- Admin portal backend (handles privileged actions: role changes, refunds, config)

\- Authorization/RBAC check (validates requested action against user's role)



\## Data stores

\- Roles \& permissions DB

\- Audit log store



\## Data flows

1\. Admin user -> Admin portal backend : requests privileged action

2\. Admin portal backend -> Admin user : returns result

3\. Admin portal backend -> Authorization check : validates request

4\. Authorization check -> Roles \& permissions DB : reads permission level

5\. Admin portal backend -> Audit log store : writes action record

6\. Regular user -> Admin portal backend : attempts privileged action directly (abuse case)



\## Trust boundaries

\- Client <-> server (both Admin user and Regular user are external/untrusted)

