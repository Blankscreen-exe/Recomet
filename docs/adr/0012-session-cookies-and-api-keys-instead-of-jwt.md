# ADR 0012: Session cookies and API keys instead of JWT

- **Status:** Accepted
- **Date:** 2026-09-11

## Context

Recomet authenticates two kinds of callers:

1. **The admin** using the web UI, which is server-rendered with Jinja2 and served from the same
   origin as the API.
2. **Machines** — integrators' backends and automation — calling the `/v1` API.

The design has requirements that any authentication scheme must meet:

- Revoking an API key must take effect immediately.
- Logout, password change and password reset must end sessions immediately.
- Integration must be as simple as possible for integrators.
- Authentication on the serving hot path must add negligible latency.
- Services are stateless and horizontally scaled, so no scheme may depend on sticky sessions.

JWT was proposed because of two concerns: that session-based auth does not scale, and that
session cookies are easy to attack.

## Decision

- **Web UI:** server-side sessions stored in Redis, identified by a random ID in a hardened
  cookie (`__Host-` prefix, `httpOnly`, `Secure`, `SameSite=Lax`), with CSRF tokens.
- **API:** opaque API keys sent as `Authorization: Bearer rk_...`, stored hashed, cached in
  process, and revoked via Redis pub/sub.
- **JWT is not used** for either path in v1.

Full session-handling measures are listed in [ARCHITECTURE.md §10.3](../ARCHITECTURE.md#103-admin-account-and-sessions).

## Why the concerns do not apply

**Scaling.** Sessions only fail to scale when held in one server's memory. Recomet stores them in
Redis, which every replica shares, so any replica validates a session with a single sub-millisecond
lookup. Session traffic is also negligible — one admin using the UI. The high-volume traffic
(serving and ingestion) uses API keys, not sessions.

**Security.** Each known attack on session cookies has a standard defence, all adopted here:

| Attack | Defence |
|---|---|
| XSS reads the cookie | `httpOnly`; strict Content Security Policy |
| CSRF | `SameSite=Lax` and CSRF tokens |
| Interception | `Secure` flag, HTTPS, HSTS |
| Session fixation | New session ID at login |
| Guessing | 256-bit random IDs |
| Theft | Server-side deletion takes effect immediately |
| Redis dump leak | Only hashes of session IDs are stored |

## Alternatives considered

### JWT for the web UI

- A JWT cannot be revoked before it expires. Meeting the revocation requirements needs a
  server-side denylist, which is a lookup per request — the same cost as a session, with more
  moving parts.
- Stored in `localStorage`, a JWT is readable by any XSS. Stored in a cookie, it has the same CSRF
  exposure as a session cookie and gains nothing over one.
- It brings JWT-specific pitfalls: `alg: none`, algorithm confusion, weak signing secrets that can
  be brute-forced offline, and payloads that are encoded but not encrypted.

Rejected.

### JWT for API access (client-credentials flow)

- Integrators would exchange a secret for a token and refresh it before expiry — more integration
  work than sending one header.
- Instant key revocation would again need a denylist.
- The claimed performance benefit does not exist: API key validation is an in-process cache hit.

Rejected. API keys are also the norm for developer-facing APIs (Stripe, OpenAI, Algolia, Meilisearch).

## Consequences

**Positive**

- Revocation is immediate for both sessions and keys.
- Integrators authenticate with a single static header.
- Fewer security pitfalls; the defences are standard and well understood.

**Negative**

- Every authenticated request requires a state lookup (Redis for sessions, an in-process cache
  for keys). This is acceptable given the volumes involved.
- Redis becomes a dependency of the admin login. It is already a hard dependency of the engine.

## When to revisit

JWT is the right tool in two planned features, and will be used there:

- **Browser-safe tokens** — the integrator's backend mints a short-lived JWT scoped to one end
  user and placement, so a browser can fetch recommendations without holding a secret key. Short
  expiry makes revocation unnecessary.
- **SSO (OIDC)** — the identity provider issues a JWT ID token, which Recomet verifies once and then
  exchanges for a normal session.

Revisit this decision if Recomet gains a dashboard served from a different origin, or a mobile client.
