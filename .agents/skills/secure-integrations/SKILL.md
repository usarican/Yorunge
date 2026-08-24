---
name: secure-integrations
description: "Security-critical paths in Yörünge — GitHub OAuth login, GitHub App webhooks, Fernet encryption of stored OAuth tokens, JWT session handling, slowapi rate limiting, CORS, and secret management. Use whenever touching backend/app/core/security.py, app/api/v1/auth.py, app/api/v1/webhooks.py, app/services/github_service.py, or anything that stores, reads, or transmits a credential."
---

# Secure Integrations (OAuth, Tokens, Webhooks)

Rules here are hard requirements from `docs/architecture/backend-architecture.md` §7. Treat a violation as a release blocker, not a style nit.

## GitHub OAuth token storage — Fernet, always

The user's GitHub access token is never persisted in plaintext.

```python
# app/core/security.py
_fernet = Fernet(settings.encryption_key.get_secret_value().encode())

def encrypt_token(raw: str) -> str:
    return _fernet.encrypt(raw.encode()).decode()

def decrypt_token(stored: str) -> str:
    return _fernet.decrypt(stored.encode()).decode()
```

- The column is `users.encrypted_github_token`. Anything writing a raw token to it is a bug.
- Decrypt at the point of use, hold it in a local variable, never put it in agent state, logs, telemetry events, SSE frames, or an API response.
- `ENCRYPTION_KEY` is a 32-byte urlsafe-base64 Fernet key from the environment. Generating one at runtime would make every stored token unreadable after restart — it must come from config.
- Key rotation, if ever needed, is a migration that decrypts with the old key and re-encrypts with the new one, not a silent key swap.

## OAuth flow

- `state` parameter is required: generate a random value, store it server-side (or in a signed short-lived cookie), and reject the callback if it does not match. Without it the login is CSRF-open.
- Request the minimum scopes needed for repo AST analysis (`read:user`, `repo` only if private repos are in scope — prefer `public_repo`).
- The `client_secret` is `SecretStr` in `settings` and only ever leaves the process in the token-exchange POST body.
- Session to the frontend is a JWT with a short expiry (`exp`), signed with a dedicated secret. Put `sub` (user id) in it — never the GitHub token.

## Webhooks

Every request to `app/api/v1/webhooks.py` is untrusted until proven otherwise:

```python
expected = "sha256=" + hmac.new(secret, raw_body, hashlib.sha256).hexdigest()
if not hmac.compare_digest(expected, request.headers.get("X-Hub-Signature-256", "")):
    raise HTTPException(status.HTTP_401_UNAUTHORIZED)
```

- Verify against the **raw** body bytes, before any JSON parsing.
- `hmac.compare_digest`, never `==`.
- Respond 2xx fast and hand the payload to an Arq job; GitHub times out at ~10s and will redeliver.
- Deliveries repeat — make handlers idempotent on `X-GitHub-Delivery` or the PR head SHA.

## Rate limiting

`slowapi` limits on every LLM-backed endpoint, keyed on user id (falling back to IP for unauthenticated routes): interview turns, quiz generation, knowledge Q&A. This protects the Anthropic bill, not just the server. A new LLM endpoint without a limiter is incomplete.

## Authorization

Every resource read/write filters on the authenticated user's id inside the query. Do not fetch by primary key and then compare ownership in Python — that pattern leaks existence through timing and through careless error paths.

## Things never to do

- Log or echo a token, secret, or full `Authorization` header. Redact to a prefix if you must trace something.
- Commit a real value to `.env.example`, `docker-compose.yml`, or a doc — placeholders only.
- Widen CORS to `allow_origins=["*"]` together with `allow_credentials=True`; list the frontend origin explicitly per environment.
- Disable TLS verification on outbound calls to "make it work".

## Verification

`pytest` must cover: signature rejection on a tampered webhook body, encrypt→decrypt round-trip, and a 401 for a cross-user resource fetch.
