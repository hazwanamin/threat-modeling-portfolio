\# Payment API - element inventory



\## External entities

\- User (browser/mobile app)

\- Payment gateway (3rd party processor)



\## Processes

\- Payment API (validates and processes transaction requests)



\## Data stores

\- Payment DB (transaction records, card metadata)

\- CyberArk vault (service account credentials, API keys)



\## Data flows

1\. User -> Payment API : submits transaction request

2\. Payment API -> Payment DB : reads/writes transaction record

3\. Payment API -> CyberArk vault : fetches service credentials

4\. Payment API -> Payment gateway : forwards transaction for processing

5\. Payment gateway -> Payment API : returns transaction result

6\. Payment API -> User : returns confirmation/receipt



\## Trust boundaries

\- Single trust boundary drawn around internal infrastructure (Payment API, Payment DB, CyberArk vault)

\- Both User and Payment gateway treated as external/untrusted, sitting outside this boundary

