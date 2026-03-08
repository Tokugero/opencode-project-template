# OWASP Top 10:2021 — Web Application Security

_Source: https://owasp.org/Top10/2021/_
_Last updated: 2026-03-08_

## A01:2021 — Broken Access Control

Access control enforces policy preventing users from exceeding intended
permissions. Failures enable unauthorized information disclosure, data
modification, or business function execution.

**What to look for:**
- Violation of least-privilege or deny-by-default
- Bypassing access checks via URL/parameter/API request tampering
- Insecure direct object references (viewing another user's data by changing an ID)
- Missing access controls on POST, PUT, DELETE operations
- Privilege escalation (acting as admin without being one)
- JWT/cookie/metadata manipulation
- CORS misconfiguration allowing untrusted origins
- Force browsing to authenticated/admin pages

**Mapped CWEs:** CWE-22, CWE-284, CWE-285, CWE-352, CWE-639, CWE-862,
CWE-863, CWE-913 (and 26 others)

---

## A02:2021 — Cryptographic Failures

Failures related to cryptography that expose sensitive data. Previously
"Sensitive Data Exposure" — the root cause is weak or missing cryptography.

**What to look for:**
- Data transmitted in cleartext (HTTP, SMTP, FTP)
- Weak or deprecated cryptographic algorithms (MD5, SHA1, DES)
- Default, weak, or reused cryptographic keys
- Missing TLS enforcement or improper certificate validation
- Passwords stored without salted hashing (or using fast hashes like MD5)
- Missing encryption at rest for sensitive data
- Insufficient randomness for cryptographic purposes

**Mapped CWEs:** CWE-261, CWE-296, CWE-310, CWE-327, CWE-328, CWE-331,
CWE-338, CWE-522

---

## A03:2021 — Injection

User-supplied data is sent to an interpreter as part of a command or query
without proper validation, filtering, or sanitization.

**What to look for:**
- String concatenation in SQL queries with user input
- User input in OS commands, LDAP queries, or XPath expressions
- Cross-site scripting (XSS) — reflected, stored, or DOM-based
- Template injection (SSTI)
- No parameterized queries or prepared statements
- Missing input validation or output encoding
- Lack of WAF or similar input filtering

**Mapped CWEs:** CWE-77, CWE-78, CWE-79, CWE-89, CWE-94, CWE-917

---

## A04:2021 — Insecure Design

Flaws in the design and architecture, not implementation bugs. Missing or
ineffective security controls that should have been part of the design.

**What to look for:**
- No threat modeling performed
- Missing rate limiting on sensitive operations
- Business logic that can be abused (unlimited retries, race conditions)
- Missing security requirements in design phase
- No separation of duties or trust boundaries
- Missing input validation at the design level

---

## A05:2021 — Security Misconfiguration

Improperly configured security settings, unnecessary features enabled,
default accounts/passwords, overly informative error messages.

**What to look for:**
- Unnecessary features, ports, services, or accounts enabled
- Default credentials unchanged
- Stack traces or overly detailed error messages exposed to users
- Missing security headers (CSP, X-Frame-Options, HSTS)
- Missing or permissive CORS configuration
- Directory listing enabled
- Software not up to date or missing security patches
- Cloud storage permissions too open (public S3 buckets)

**Mapped CWEs:** CWE-2, CWE-11, CWE-13, CWE-15, CWE-16, CWE-388, CWE-756

---

## A06:2021 — Vulnerable and Outdated Components

Using libraries, frameworks, or software with known vulnerabilities.

**What to look for:**
- Dependencies with known CVEs
- Unsupported or end-of-life software versions
- No regular dependency scanning or update process
- Lock files not reviewed for vulnerable transitive dependencies
- No software bill of materials (SBOM)

---

## A07:2021 — Identification and Authentication Failures

Weak authentication mechanisms that allow attackers to compromise passwords,
keys, or session tokens.

**What to look for:**
- Credential stuffing or brute force not prevented
- Weak or default passwords permitted
- Missing or ineffective multi-factor authentication
- Session IDs in URLs
- Session tokens not rotated after login
- Passwords stored in plaintext or weakly hashed
- Missing account lockout after repeated failures

**Mapped CWEs:** CWE-255, CWE-259, CWE-287, CWE-288, CWE-384, CWE-798

---

## A08:2021 — Software and Data Integrity Failures

Code and infrastructure that do not protect against integrity violations:
unsigned updates, untrusted CI/CD pipelines, insecure deserialization.

**What to look for:**
- Dependencies from untrusted sources without integrity verification
- Unsigned or unverified software updates (auto-update without signing)
- Insecure CI/CD pipeline (code injection into build process)
- Insecure deserialization of untrusted data
- Missing subresource integrity (SRI) for CDN-hosted assets

**Mapped CWEs:** CWE-345, CWE-353, CWE-426, CWE-494, CWE-502, CWE-565,
CWE-784, CWE-829, CWE-830, CWE-915

---

## A09:2021 — Security Logging and Monitoring Failures

Insufficient logging, detection, monitoring, and active response. Without
these, breaches go undetected.

**What to look for:**
- Login failures, access control failures, and input validation failures
  not logged
- Log messages lacking sufficient context (who, what, when, where)
- Logs stored only locally with no alerting
- No monitoring for suspicious activity patterns
- Penetration testing and DAST scans not triggering alerts
- Application unable to detect or alert on active attacks in real time

---

## A10:2021 — Server-Side Request Forgery (SSRF)

The application fetches a remote resource based on user-supplied input
without validating the destination URL.

**What to look for:**
- URLs accepted as parameters that fetch remote resources
- Internal/private network addresses accessible via SSRF
- Cloud metadata endpoints reachable (169.254.169.254)
- File:// or other protocol handlers accepted
- No allowlist for outbound request destinations
- Redirect chains that reach internal services

**Mapped CWEs:** CWE-918
