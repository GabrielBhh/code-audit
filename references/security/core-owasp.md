# Core OWASP — Always Load

Universal security concerns that apply regardless of language or context.

## A01 — Broken Access Control
- Direct object references using user-supplied IDs without authorization checks
- Missing role/permission checks before sensitive operations
- Path traversal: `../` in file paths derived from user input
- **Resource ownership not verified after auth** — JWT validated but resource ownership not checked (e.g. `GET /projects/{id}` returns data without verifying `project.owner_id == current_user.id`)
- **Privileged endpoint missing admin/role guard** — settings, API-key management, admin routes with only auth but no role check

## A02 — Cryptographic Failures
- HTTP URLs for sensitive data transmission
- Weak algorithms: MD5, SHA1, DES, RC4 for passwords or tokens
- Hardcoded encryption keys
- Missing TLS certificate validation (`verify=False`, `InsecureSkipVerify`)
- Storing passwords in plaintext or with reversible encoding
- **Non-constant-time secret comparison** — vulnerable to timing attacks; use `hmac.compare_digest` or equivalent

## A04 — Insecure Design
- Business logic that can be abused (e.g., negative quantities, skipping steps)
- Missing rate limiting on authentication or sensitive endpoints
- **File content read fully into memory before size check** — attacker sends a huge payload; check `Content-Length` first or stream with a size cap
- **Unconstrained string field where an enum is required** — use `Literal[...]` or `Enum`

## A05 — Security Misconfiguration
- Debug mode enabled in production (`DEBUG=True`, `app.run(debug=True)`)
- Default credentials not changed
- Verbose error messages exposing stack traces to end users
- Unnecessary open ports or services

## A06 — Vulnerable & Outdated Components
- Newly added dependencies with known CVEs
- Pinning to a specific old version with known vulnerabilities
- **Known unmaintained packages** — `python-jose` (no releases since 2021), `pycrypto` (abandoned), `node-uuid`, `request` (deprecated)

## A07 — Identification & Authentication Failures
- Weak password policies
- Missing account lockout after failed attempts
- JWT without expiry (`exp` claim missing)
- Sessions not invalidated on logout
- **Hardcoded fallback secret** — `os.getenv("SECRET_KEY", "change-me-in-prod")` — fail loudly: `os.environ["SECRET_KEY"]`

## A08 — Software & Data Integrity Failures
- Deserialization of untrusted data (`pickle.loads`, `yaml.load` without `Loader=`)
- Missing integrity checks on downloaded artifacts

## A09 — Security Logging & Monitoring Failures
- Authentication failures not logged
- Sensitive operations (admin actions, privilege changes) not audited

## A10 — Server-Side Request Forgery (SSRF)
- User-controlled URLs passed to HTTP clients without validation
- Internal endpoints reachable via redirect
