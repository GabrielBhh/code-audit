# API Authentication Security

Load when auth endpoints, JWT handling, or session code is detected.

## JWT

- **JWT passed as URL query parameter** — `?token=<jwt>` gets recorded in server logs, proxy logs, browser history. Use `Authorization: Bearer` header instead
- **Token stored in `localStorage`** — accessible to any JavaScript on the page (XSS risk). Use `httpOnly` + `Secure` + `SameSite=Strict` cookies
- **JWT without expiry** — `exp` claim missing allows infinite-lifetime tokens
- **JWT algorithm not pinned** — `jwt.decode(token, key)` without `algorithms=["HS256"]` allows `"alg": "none"` confusion attacks
- **Refresh token not rotated** — issuing new access tokens without invalidating the old refresh token enables replay after compromise

## Session / Cookie Auth

- Sessions not invalidated on logout
- Missing `Secure`, `HttpOnly`, `SameSite` flags on auth cookies
- Session fixation: session ID not regenerated on login

## Password Handling

- Weak password policies
- Missing account lockout after N failed attempts
- Plaintext or reversibly-encoded storage
- Weak hashing: MD5, SHA1, plain SHA256 → use `bcrypt`, `scrypt`, or `argon2`

## OAuth / Social Login

- State parameter missing on OAuth redirect (CSRF)
- Redirect URI not on allowlist
- Access token logged or leaked in referer header
