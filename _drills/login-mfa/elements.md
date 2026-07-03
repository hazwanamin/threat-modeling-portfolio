# Login with MFA — element inventory

## External entities
- User (browser/mobile app)

## Processes
- Auth service (validates username/password)
- MFA verification service (validates OTP/push approval)

## Data stores
- User credentials DB (password hashes)
- MFA secrets store (TOTP seeds / push tokens)
- Session store (issued session tokens/JWTs)

## Data flows
1. User -> Auth service : submits username/password
2. Auth service -> User credentials DB : checks password hash
3. Auth service -> MFA verification service : triggers OTP challenge
4. User -> MFA verification service : submits OTP
5. MFA verification service -> MFA secrets store : validates OTP against seed
6. MFA verification service -> Session store : writes session on success
7. Session store -> User : returns session token

## Trust boundaries
- Client <-> server (User is untrusted; everything else is internal infrastructure)