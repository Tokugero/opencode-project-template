# OWASP API Security Top 10:2023

_Source: https://owasp.org/API-Security/editions/2023/en/0x11-t10/_
_Last updated: 2026-03-08_

## API1:2023 — Broken Object Level Authorization

APIs expose endpoints handling object identifiers, creating a wide attack
surface for unauthorized data access when authorization checks are missing.

**What to look for:**
- Endpoints that accept user-supplied IDs without ownership verification
- Missing authorization checks when accessing data by ID parameter
- No per-object permission validation (user A accessing user B's resource)
- GraphQL queries that traverse relationships without auth checks

---

## API2:2023 — Broken Authentication

Authentication mechanisms implemented incorrectly, allowing attackers to
steal tokens or impersonate users.

**What to look for:**
- Weak token generation or predictable session identifiers
- Credentials transmitted over unencrypted channels
- JWT tokens without proper signature validation or with weak algorithms
- Missing token expiration or rotation
- Hardcoded credentials or API keys in source code
- No rate limiting on authentication endpoints

---

## API3:2023 — Broken Object Property Level Authorization

Missing or improper authorization at the object property level, enabling
unauthorized information exposure or data modification.

**What to look for:**
- APIs returning sensitive fields to unauthorized users (over-fetching)
- Mass assignment vulnerabilities (client can set admin, role, or
  internal-only properties via PUT/PATCH)
- No field-level access control on responses
- GraphQL schemas exposing internal fields without authorization

---

## API4:2023 — Unrestricted Resource Consumption

Missing controls on API resource consumption enabling denial of service
or excessive operational costs.

**What to look for:**
- Missing rate limiting per user/API key
- No pagination enforcement (unlimited result sets)
- Unthrottled file uploads (no size/count limits)
- Expensive operations without resource budgets
- Missing timeout on downstream service calls
- Batch endpoints without item count limits

---

## API5:2023 — Broken Function Level Authorization

Complex access control with unclear admin/user separation causing
authorization flaws on function-level operations.

**What to look for:**
- Administrative functions accessible to regular users
- Role-based checks missing from sensitive endpoints
- Inconsistent permission validation across similar endpoints
- PUT/DELETE methods on resources where user should only GET
- Admin endpoints guessable by URL pattern (e.g., /api/admin/*)

---

## API6:2023 — Unrestricted Access to Sensitive Business Flows

APIs expose business-critical flows without compensating controls for
automated or excessive use.

**What to look for:**
- No velocity checks on sensitive operations (purchases, transfers)
- Missing bot detection on business-critical flows
- No account-based throttling on high-value operations
- Automated abuse possible (ticket scalping, mass registration)
- Missing CAPTCHA or proof-of-work on abuse-prone endpoints

---

## API7:2023 — Server Side Request Forgery (SSRF)

API fetches a remote resource based on user-supplied URI without validating
the destination, enabling requests to internal services.

**What to look for:**
- APIs accepting URLs as parameters for fetch/webhook/callback
- No allowlist validation on outbound request destinations
- Internal/private network addresses reachable via API
- Redirect following without destination validation
- Cloud metadata endpoints accessible (169.254.169.254)

---

## API8:2023 — Security Misconfiguration

Complex API and infrastructure configurations left insecure through
oversight or default settings.

**What to look for:**
- Debug endpoints left enabled in production
- Unnecessary HTTP methods allowed (TRACE, OPTIONS returning too much)
- Overly permissive CORS policies (Access-Control-Allow-Origin: *)
- Missing security headers
- Default credentials on API management tools
- Verbose error messages exposing stack traces or internal details
- TLS misconfiguration or missing encryption

---

## API9:2023 — Improper Inventory Management

Undocumented, deprecated, or forgotten API endpoints remain active and
create an unmanaged attack surface.

**What to look for:**
- Deprecated API versions still responding to requests
- Beta/staging/development endpoints accessible in production
- Undocumented endpoints (not in OpenAPI spec)
- Old API versions with known vulnerabilities still active
- Missing API gateway routing rules (catchall routes)
- Shadow APIs created by individual teams without central tracking

---

## API10:2023 — Unsafe Consumption of APIs

Developers trust data from third-party APIs more than user input, applying
weaker security standards to integrated services.

**What to look for:**
- Missing validation of third-party API responses
- No rate limiting on calls to external services
- Unencrypted communication with third-party APIs
- No error handling for malformed third-party responses
- Blindly trusting redirects from external services
- Processing third-party webhook payloads without signature verification
