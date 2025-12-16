# Security and Permissions

[Index](../README.md) | [Previous: Pragmatism](02-pragmatism.md)

---

Security for developers is one of the toughest subjects to understand. Keep things simple and include plenty of diagrams with concepts in your documentation. Do not try to invent something new with security; always use tried and true patterns that can be easily understood.

## Security Principles

Before diving into specifics, understand the foundational principles that guide security decisions.

### Zero Trust

"Never trust, always verify." Zero Trust[^1] assumes that threats exist both outside and inside the network. Every request must be authenticated and authorised, regardless of where it originates.

**Core tenets:**

- Verify explicitly. Always authenticate and authorise based on all available data points.
- Use least privilege access. Limit user access with just-in-time and just-enough-access.
- Assume breach. Minimise blast radius and segment access. Verify end-to-end encryption.

Zero Trust is not a product you buy. It's an architecture approach that affects how you design every API interaction. See Microsoft's Zero Trust guidance[^2] and NIST SP 800-207[^3] for detailed implementation frameworks.

### Defense in Depth

Layer your security controls. Don't rely on a single mechanism. If one layer fails, others should still protect the system. This principle is documented in NIST's cybersecurity framework[^4].

```
┌─────────────────────────────────────────┐
│            API Gateway                  │  ← Rate limiting, WAF
│  ┌─────────────────────────────────┐    │
│  │       Authentication            │    │  ← Token validation
│  │  ┌─────────────────────────┐    │    │
│  │  │    Authorisation        │    │    │  ← Permission checks
│  │  │  ┌──────────────────┐   │    │    │
│  │  │  │ Input Validation │   │    │    │  ← Schema validation
│  │  │  │  ┌───────────┐   │   │    │    │
│  │  │  │  │  Business │   │   │    │    │  ← Domain rules
│  │  │  │  │   Logic   │   │   │    │    │
│  │  │  │  └───────────┘   │   │    │    │
│  │  │  └──────────────────┘   │    │    │
│  │  └─────────────────────────┘    │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```

Each layer catches different types of threats.

### CIA Triad

The three pillars of information security, as defined by ISO/IEC 27001[^5]:

| Principle | Definition | API Implications |
|-----------|------------|------------------|
| **Confidentiality** | Data is accessible only to authorised parties | Encryption in transit (TLS[^6]), proper authorisation, field-level access control |
| **Integrity** | Data is accurate and unaltered | Input validation, checksums, audit logs, signed payloads |
| **Availability** | Systems are accessible when needed | Rate limiting, circuit breakers, DDoS protection, redundancy |

When designing security controls, consider which pillar you're protecting. Sometimes they conflict: aggressive rate limiting (availability) might make legitimate access harder (also availability). Find the right balance.

### Fail Secure

When something goes wrong, fail closed rather than open. An authentication service that crashes should deny access, not grant it. See OWASP's guidance on secure defaults[^7].

- **Database unavailable?** Deny requests. Don't skip authorisation.
- **Token validation fails?** Treat as unauthenticated.
- **Rate limit service down?** Apply conservative defaults, don't allow unlimited access.
- **Configuration missing?** Use secure defaults, not permissive ones.

This is harder than it sounds. The temptation is to "keep things working" but insecure working is worse than secure failing.

### Separation of Duties

No single person or service should have complete control over critical operations. Break sensitive operations into steps that require multiple parties. This is a core principle in SOC 2 compliance[^8].

- Deployment to production requires code review approval
- Database admin cannot also be application admin
- Secret rotation requires approval from security team
- Privileged access requires justification and time-limited grants

For APIs, this means service accounts should have focused permissions, not "god mode" access.

## Authentication vs Authorisation

These are different concerns:

- **Authentication** - Who are you? Verifying identity (login, credentials, tokens)
- **Authorisation** - What can you do? Checking permissions after identity is established

**OAuth 2.0[^9] is an authorisation framework, not authentication.** It grants access to resources but doesn't verify identity. OpenID Connect (OIDC)[^10] builds on OAuth 2.0 to add authentication. When you "Login with Google," that's OIDC providing identity, with OAuth handling the access grants.

For APIs:

- **Private/internal APIs** - JWT[^11] tokens for stateless auth
- **Public-facing APIs** - OIDC for authentication + OAuth 2.0 for authorisation (use PKCE[^12] for mobile/SPA clients)
- **Avoid Basic Auth** - It's a standard for user authentication but not appropriate for application-to-application communication. See RFC 7617[^13].

If you have to store credentials, never store the password. Use a dedicated password hashing algorithm designed for this purpose.

## Principals and Subjects

1. Use Role-Based Access Control (RBAC)[^14]
2. Apply the **Principle of Least Privilege**[^15] (PoLP): grant only the minimum permissions required for the task
3. Keep roles as simple as possible. They always become more complex as time evolves
4. Use a **grant-based** permissions model, never restriction-based. Start with no access and explicitly grant what's needed
5. Always add rate limiting to endpoints. Include rate limit status in response headers

**PoLP Example:**

A reporting service needs to read order data. The wrong approach grants broad access:

```
# Bad - too permissive
Role: reporting-service
Permissions: orders.*, customers.*, products.*
```

The right approach grants only what's needed:

```
# Good - minimal required access
Role: reporting-service
Permissions: orders.read, orders.list
Restrictions: only orders older than 24 hours
```

If the reporting service is compromised, the attacker can only read historical orders. They can't modify orders, access customers, or see real-time data. The blast radius is contained.

## Circles of Trust

Trust is not binary. Systems operate within concentric circles of trust, where inner circles have higher trust levels than outer circles.

```
┌─────────────────────────────────────────────────────────┐
│                    Public Internet                      │
│   ┌─────────────────────────────────────────────────┐   │
│   │              Partner Network                    │   │
│   │   ┌─────────────────────────────────────────┐   │   │
│   │   │           Corporate Network             │   │   │
│   │   │   ┌─────────────────────────────────┐   │   │   │
│   │   │   │        Service Mesh             │   │   │   │
│   │   │   │   ┌─────────────────────────┐   │   │   │   │
│   │   │   │   │    Internal Services    │   │   │   │   │
│   │   │   │   └─────────────────────────┘   │   │   │   │
│   │   │   └─────────────────────────────────┘   │   │   │
│   │   └─────────────────────────────────────────┘   │   │
│   └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

**Trust levels and their implications:**

| Circle | Trust Level | Authentication Required | Example |
|--------|-------------|------------------------|---------|
| Public Internet | None | Full auth + rate limiting | Mobile app users |
| Partner Network | Low | API keys + mutual TLS[^16] | Third-party integrations |
| Corporate Network | Medium | SSO + network controls | Internal tools |
| Service Mesh | High | Service identity (mTLS) | Microservices |
| Internal Services | Highest | Implicit trust | Same-process calls |

**Key principles:**

1. **Never trust based on network location alone.** VPNs get compromised. Assume breach.
2. **Authenticate at every boundary crossing.** When a request moves from one circle to another, re-verify.
3. **Least privilege still applies.** Even trusted internal services should only access what they need.
4. **Trust degrades over time.** Session tokens from high-trust contexts should expire faster when used in lower-trust contexts.

**Federation across trust boundaries:**

When systems in different trust circles need to communicate, use federation protocols:

- **SAML 2.0**[^17] for enterprise SSO across organisations
- **OIDC**[^10] for identity federation with external providers
- **OAuth 2.0**[^9] for delegated access across trust boundaries

The identity provider acts as a trust broker, vouching for identities across circles.

## Passwords

If you have to store passwords, use a proper password hashing algorithm:

1. **Argon2id**[^18] - The current recommendation, winner of the Password Hashing Competition[^19]
2. **bcrypt**[^20] - Battle-tested and widely available
3. **scrypt**[^21] - Good alternative with memory-hard properties

Never use general-purpose hash functions (MD5, SHA-256) for passwords. These are too fast and vulnerable to brute-force attacks. Password hashing algorithms are intentionally slow and include built-in salting. See OWASP Password Storage Cheat Sheet[^22].

```javascript
// Example using bcrypt (Node.js)
const bcrypt = require('bcrypt');
const saltRounds = 12;

// Hashing
const hash = await bcrypt.hash(password, saltRounds);

// Verification
const match = await bcrypt.compare(password, hash);
```

## One-Time Passwords (OTP)

OTPs provide a second authentication factor. Something the user knows (password) plus something the user has (device generating OTP). See NIST SP 800-63B[^23] for authentication assurance levels.

### TOTP (Time-Based One-Time Password)

TOTP[^24] generates codes based on current time and a shared secret. Codes are valid for a short window (typically 30 seconds).

```
Code = HMAC-SHA1(secret, floor(timestamp / 30)) → 6 digits
```

**Advantages:**

- Works offline (no network required)
- Standardised (RFC 6238)
- Supported by authenticator apps (Google Authenticator[^25], Authy[^26], 1Password[^27])

**Implementation notes:**

- Store the shared secret encrypted, not plaintext
- Allow for clock drift (accept codes from adjacent time windows)
- Provide backup codes for account recovery
- Rate limit verification attempts

### HOTP (HMAC-Based One-Time Password)

HOTP[^28] generates codes based on a counter rather than time. Each code is valid until used.

```
Code = HMAC-SHA1(secret, counter) → 6 digits
```

Less common than TOTP. Used in hardware tokens where time synchronisation is difficult.

### SMS and Email OTP

Sending codes via SMS or email is convenient but less secure:

| Method | Pros | Cons |
|--------|------|------|
| SMS | Familiar to users, no app needed | SIM swapping attacks[^29], SS7 vulnerabilities[^30], delivery delays |
| Email | No phone needed | Email accounts get compromised, delivery delays |
| Authenticator App | Most secure, works offline | Requires app installation, device dependency |

**If you must use SMS OTP:**

- Use it as a fallback, not the primary method
- Implement rate limiting aggressively
- Monitor for SIM swap indicators (sudden carrier changes)
- Consider voice calls as an alternative

### OTP Best Practices

1. **Short expiry** - TOTP: 30-60 seconds. SMS/Email: 5-10 minutes max
2. **Single use** - Once verified, invalidate the code
3. **Rate limiting** - Max 3-5 attempts per code, then require new code
4. **Constant-time comparison** - Prevent timing attacks[^31] when verifying
5. **Secure delivery** - For SMS/Email, don't include what the code is for ("Your code is 123456" not "Your password reset code is 123456")

```javascript
// Constant-time comparison to prevent timing attacks
function verifyOTP(provided, expected) {
  if (provided.length !== expected.length) return false;
  let result = 0;
  for (let i = 0; i < provided.length; i++) {
    result |= provided.charCodeAt(i) ^ expected.charCodeAt(i);
  }
  return result === 0;
}
```

## Encrypted Transmission

Use TLS 1.3[^6] everywhere, no exceptions. Don't worry about debugging the payload. Tools like Charles Proxy[^32], mitmproxy[^33], or browser developer tools can help you inspect traffic during development. See Mozilla's TLS configuration recommendations[^34].

## Tokens

Use JWT (JSON Web Tokens)[^11] for stateless authentication. JWTs have been around for years, are easy to explain, and the internet is rich with examples. See jwt.io[^35] for an interactive debugger.

JWT provides access to claims. You can include:

- **Version number** - Notify clients when the app version is outdated
- **Roles** - Allow clients to adjust presentation based on permissions
- **Subject name** - Display user information without additional API calls

Claims can be decoded (they're Base64-encoded), providing metadata to the client. However, never put information in the JWT that could compromise the user account if exposed.

### Signature Verification

**Always verify JWT signatures.** This sounds obvious, but it's a common vulnerability. Never trust a token just because it parses correctly. See Auth0's JWT vulnerabilities guide[^36].

Critical rules:

- **Declare your algorithm explicitly** - Never trust the `alg` header in the token. Attackers can set `alg: none` or switch from RS256 to HS256 to bypass verification. Your server must enforce which algorithm it accepts. See CVE-2015-9235[^37].
- **Validate the issuer (`iss`)** - Ensure the token came from your identity provider
- **Check expiration (`exp`)** - Reject expired tokens
- **Check not-before (`nbf`)** - Reject tokens that aren't yet valid. Prevents tokens issued for future use from being used early.
- **Verify audience (`aud`)** - Ensure the token was intended for your API

```javascript
// Bad - trusts the token's algorithm claim
jwt.verify(token, secret);

// Good - explicitly declares expected algorithm
jwt.verify(token, publicKey, { algorithms: ['RS256'], issuer: 'https://auth.example.com' });
```

### JTI and Device Binding

The `jti` (JWT ID) claim is a unique identifier for each token. Use it to:

- **Prevent replay attacks** - Track which tokens have been used
- **Enable revocation** - Invalidate specific tokens before expiry
- **Bind tokens to devices** - Associate JTI with device fingerprints

For distributed systems, store JTI state in Redis[^38] or similar:

```
jti:abc123 -> { deviceFingerprint: "x1y2z3", issuedAt: 1699876543, revoked: false }
```

On each request:

1. Extract JTI from token
2. Look up JTI in Redis
3. Verify device fingerprint matches the request
4. Reject if revoked or fingerprint mismatch

This catches token theft; even with a valid token, an attacker's device fingerprint won't match. See [Rate Limiting Keys](#rate-limiting-keys) for how to combine JTI tracking with rate limiting in a single Redis lookup.

### Token Storage

**Never store JWTs in localStorage or sessionStorage.** These are accessible to any JavaScript running on the page, making them vulnerable to XSS attacks[^39].

Instead, use HTTP-only cookies:

```
Set-Cookie: access_token=eyJ...; HttpOnly; Secure; SameSite=Strict; Path=/
```

- **HttpOnly** - JavaScript cannot access the cookie
- **Secure** - Only sent over HTTPS
- **SameSite=Strict**[^40] - Prevents CSRF by not sending on cross-origin requests

### Access and Refresh Tokens

Use a two-token pattern as described in OAuth 2.0 Security Best Practices[^41]:

| Token | Lifetime | Storage | Purpose |
|-------|----------|---------|---------|
| Access token | 15-20 minutes | HTTP-only cookie or memory | Authorise API requests |
| Refresh token | Days/weeks | HTTP-only cookie (separate path) | Obtain new access tokens |

The refresh flow:

1. Access token expires
2. Client calls `/auth/refresh` with refresh token
3. Server validates refresh token, issues new access token
4. Old refresh token is invalidated (rotation)

Keep refresh tokens on a separate path (`Path=/auth/refresh`) so they're only sent to the refresh endpoint.

### Risk-Based Assessment (RBA)

Not all authenticated requests should be treated equally. A user reading their profile is low risk. A user changing their password or email is high risk. Risk-Based Assessment evaluates context and adjusts security requirements accordingly. See NIST SP 800-63B[^23] for guidance on adaptive authentication.

**Building a user behaviour baseline:**

RBA requires tracking user interactions over time to establish what's "normal" for each user. Store this in your database:

```
user_sessions: {
  userId: "user_123",
  knownDevices: ["fp_abc123", "fp_def456"],
  knownIPs: ["203.0.113.0/24"],
  knownLocations: ["Sydney, AU", "Melbourne, AU"],
  typicalHours: { start: 7, end: 22, timezone: "Australia/Sydney" },
  lastLogin: "2024-01-15T09:30:00Z",
  loginHistory: [{ timestamp, device, ip, location, success }],
  failedAttempts: { count: 0, lastAttempt: null }
}
```

**Risk signals to evaluate:**

- New device or browser fingerprint
- New geographic location or IP address
- Unusual time of day for this user
- Multiple failed attempts recently
- Velocity (too many requests in short time)
- Value of the action (viewing vs transferring money)

**Scoring the request:**

Assign points to each risk signal and sum them:

| Signal | Points |
|--------|--------|
| Known device | 0 |
| New device, known location | +10 |
| New device, new location | +30 |
| New country | +50 |
| Outside typical hours | +10 |
| Failed attempts > 3 in last hour | +20 |
| First-time login (new account) | +25 |
| Sensitive operation | +20 |
| Impossible travel (Sydney to London in 1 hour) | +100 |

Then map total score to response:

- **0-20**: Allow
- **21-50**: Challenge (MFA)
- **51+**: Block or require enhanced verification

**Handling first-time interactions:**

New users and new devices have no baseline. Handle this by:

1. **New accounts** - Require email/phone verification before allowing sensitive operations
2. **New devices** - Send notification to known email: "New login from Chrome on Windows"
3. **First login from location** - Challenge with MFA, then add to known locations if successful
4. **Grace period** - For the first 7-14 days, apply stricter thresholds until baseline is established

The goal is to make low-risk actions frictionless for established users while adding friction when something looks unusual.

**Response levels:**

1. **Allow** - Low risk, proceed normally
2. **Challenge** - Medium risk, require step-up authentication (MFA, re-enter password)
3. **Block** - High risk, deny and alert

**Sensitive operations that should always require step-up authentication:**

- Password changes
- Email or phone number changes
- Adding or changing payment methods
- Enabling/disabling MFA
- Deleting account
- Exporting personal data
- Granting elevated permissions

```http
POST /users/123/change-password
Authorization: Bearer <access_token>
X-Step-Up-Token: <recent_mfa_token>

{
  "currentPassword": "...",
  "newPassword": "..."
}
```

Even with a valid session, require the user to re-authenticate for these operations. The step-up token should be short-lived (5-10 minutes) and single-use.

**Password change flow example:**

1. User clicks "Change Password"
2. API returns `403` with `X-Step-Up-Required: true`
3. Client prompts for current password or MFA
4. Client exchanges credentials for step-up token
5. Client retries with step-up token
6. API validates step-up token is recent and unused
7. Password change proceeds

This prevents attackers with stolen session tokens from locking users out of their accounts or redirecting password resets to attacker-controlled emails.

### JWS vs JWE

- **JWS**[^42] (JSON Web Signature) - Signed but readable. Claims are Base64-encoded, anyone can decode them. Use when claims aren't sensitive.
- **JWE**[^43] (JSON Web Encryption) - Encrypted payload. Only the server can read claims. Use when tokens contain sensitive data or you don't want clients inspecting claims.

Most APIs use JWS. Consider JWE if you're passing tokens between services and want to prevent intermediate systems from reading claims.

### Perimeter vs Distributed Security

**Perimeter security** assumes everything inside the network is trusted. Authenticate at the edge (API gateway), then internal services trust each other. Simple but vulnerable if the perimeter is breached.

**Distributed security** (zero trust) assumes nothing is trusted. Every service validates tokens and enforces authorisation. More resilient but requires consistent identity propagation.

For APIs, distributed security means:

- Pass identity tokens between services
- Each service validates and authorises independently
- Use service-to-service authentication (mTLS[^16], service accounts)

### IAM, Identity, and PAM

**Identity Management** is the foundation. You need a central source of truth for who users are, what groups they belong to, and what attributes they carry. In enterprises this is typically Active Directory[^44] or Azure AD. For modern applications, identity providers like Keycloak[^45], Okta[^46], or Auth0[^47] give you user management, federation, and social login out of the box.

**IAM** (Identity and Access Management) builds on identity to answer: what can this user do? IAM systems manage:

- Authentication policies (MFA, passwordless, SSO)
- Authorisation rules (roles, permissions, policies)
- Federation and trust relationships between systems

OAuth 2.0[^9] and OpenID Connect[^10] are the protocols that tie this together. OAuth handles authorisation (what can you access), OIDC adds identity (who are you). Your API validates tokens issued by the identity provider; it doesn't manage users directly.

**PAM** (Privileged Access Management) is critical and often overlooked. PAM controls access to sensitive systems: database credentials, admin consoles, production infrastructure. Key capabilities:

- **Just-in-time access** - Elevated privileges granted temporarily, not permanently
- **Credential vaulting** - Secrets stored and rotated automatically, never exposed to humans
- **Session recording** - Full audit trail of privileged actions
- **Approval workflows** - Require sign-off before granting sensitive access

Tools like HashiCorp Vault[^48], CyberArk[^49], or AWS Secrets Manager[^50] handle credential management. For API development, this means your services retrieve database passwords and API keys from a vault at runtime, not from config files or environment variables baked into deployments.

Don't build identity infrastructure yourself. Integrate with established systems and focus on your domain.

## Rate Limiting

Rate limiting prevents abuse, protects against DoS attacks, and ensures fair resource allocation. See the IETF Rate Limit Headers draft[^51] for standardised headers.

Use standard headers to communicate rate limit status:

```
RateLimit-Limit: 100
RateLimit-Remaining: 45
RateLimit-Reset: 1699876543
```

When rate limited, return `429 Too Many Requests` with a `Retry-After` header per RFC 6585[^52].

### Rate Limiting Keys

What you rate limit by matters:

- **IP address** - Simple but problematic behind NAT/proxies (entire offices share one IP)
- **User ID** - Fair per-user limits, but requires authentication
- **JTI + Device fingerprint** - Granular control per session/device

For authenticated APIs, tie rate limiting to the same Redis[^38] store as your JTI tracking. One lookup gives you token validation, device binding, and rate limit state:

```
session:{jti} -> {
  userId: "user_123",
  deviceFingerprint: "x1y2z3",
  requests: 47,
  windowStart: 1699876000,
  revoked: false
}
```

### Device Fingerprinting Challenges

Device fingerprinting is useful but imperfect. See FingerprintJS[^53] for implementation approaches.

- **Browser fingerprints drift** - Updates, extensions, and settings changes alter fingerprints over time. Use fuzzy matching, not exact equality.
- **Privacy tools defeat fingerprinting** - Tor[^54], VPNs, and privacy browsers intentionally obscure fingerprints. Decide if this blocks access or just increases scrutiny.
- **Mobile fingerprints are less stable** - App updates, OS updates, and carrier changes affect device IDs.
- **Fingerprinting has legal implications** - GDPR[^55] and similar regulations may consider fingerprints personal data.

Treat fingerprints as one signal among many, not absolute truth. Combine with other factors: IP geolocation, request patterns, time of day.

### Session Management

Decide your session policy upfront:

**Single session only** - One active session per user. New login invalidates previous sessions. Simpler, more secure, but frustrating for users with multiple devices.

```javascript
// On new login, revoke all existing sessions for this user
await redis.del(`sessions:user:${userId}:*`);
await redis.set(`sessions:user:${userId}:${newJti}`, sessionData);
```

**Multiple sessions allowed** - Users can be logged in on phone, tablet, and desktop simultaneously. Better UX, but requires:

- Session listing UI ("you're logged in on 3 devices")
- Individual session revocation
- Maximum session limits (e.g., no more than 5 active sessions)

**Concurrent session limits** - Allow N simultaneous sessions. New login beyond the limit either fails or evicts the oldest session.

For high-security applications (banking, healthcare), consider single session with step-up authentication for sensitive operations. See OWASP Session Management Cheat Sheet[^56].

---

## References

[^1]: NIST. (2020). "Zero Trust Architecture." https://www.nist.gov/publications/zero-trust-architecture
[^2]: Microsoft. "Zero Trust Guidance." https://learn.microsoft.com/en-us/security/zero-trust/
[^3]: Rose, S. et al. (2020). "Zero Trust Architecture." NIST Special Publication 800-207. https://csrc.nist.gov/publications/detail/sp/800-207/final
[^4]: NIST. "Cybersecurity Framework." https://www.nist.gov/cyberframework
[^5]: ISO. "ISO/IEC 27001 Information Security Management." https://www.iso.org/standard/27001
[^6]: Rescorla, E. (2018). "The Transport Layer Security (TLS) Protocol Version 1.3." RFC 8446, IETF. https://datatracker.ietf.org/doc/html/rfc8446
[^7]: OWASP. "Proactive Controls C6: Implement Digital Identity." https://owasp.org/www-project-proactive-controls/v3/en/c6-implement-security-defenses
[^8]: AICPA. "SOC 2 - SOC for Service Organizations: Trust Services Criteria." https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/aaborgunits/soc2
[^9]: OAuth 2.0. "OAuth 2.0 Authorization Framework." https://oauth.net/2/
[^10]: OpenID Foundation. "OpenID Connect." https://openid.net/connect/
[^11]: Jones, M., Bradley, J., and Sakimura, N. (2015). "JSON Web Token (JWT)." RFC 7519, IETF. https://datatracker.ietf.org/doc/html/rfc7519
[^12]: Sakimura, N. et al. (2015). "Proof Key for Code Exchange by OAuth Public Clients." RFC 7636, IETF. https://datatracker.ietf.org/doc/html/rfc7636
[^13]: Reschke, J. (2015). "The 'Basic' HTTP Authentication Scheme." RFC 7617, IETF. https://datatracker.ietf.org/doc/html/rfc7617
[^14]: NIST. "Role Based Access Control." https://csrc.nist.gov/projects/role-based-access-control
[^15]: NIST. "Least Privilege." NIST Glossary. https://csrc.nist.gov/glossary/term/least_privilege
[^16]: Cloudflare. "What is Mutual TLS (mTLS)?" https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/
[^17]: OASIS. "Security Assertion Markup Language (SAML) V2.0." https://wiki.oasis-open.org/security/FrontPage
[^18]: Biryukov, A., Dinu, D., and Khovratovich, D. "Argon2." https://github.com/P-H-C/phc-winner-argon2
[^19]: Password Hashing Competition. https://www.password-hashing.net/
[^20]: Wikipedia. "bcrypt." https://en.wikipedia.org/wiki/Bcrypt
[^21]: Percival, C. "scrypt." https://www.tarsnap.com/scrypt.html
[^22]: OWASP. "Password Storage Cheat Sheet." https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
[^23]: Grassi, P. et al. (2017). "Digital Identity Guidelines: Authentication and Lifecycle Management." NIST Special Publication 800-63B. https://pages.nist.gov/800-63-3/sp800-63b.html
[^24]: M'Raihi, D. et al. (2011). "TOTP: Time-Based One-Time Password Algorithm." RFC 6238, IETF. https://datatracker.ietf.org/doc/html/rfc6238
[^25]: Google. "Google Authenticator." https://support.google.com/accounts/answer/1066447
[^26]: Twilio. "Authy." https://authy.com/
[^27]: AgileBits. "1Password." https://1password.com/
[^28]: M'Raihi, D. et al. (2005). "HOTP: An HMAC-Based One-Time Password Algorithm." RFC 4226, IETF. https://datatracker.ietf.org/doc/html/rfc4226
[^29]: FBI. "SIM Swapping." https://www.fbi.gov/how-we-can-help-you/safety-resources/scams-and-safety/common-scams-and-crimes/sim-swapping
[^30]: Wired. (2016). "The Critical Hole at the Heart of Cell Phone Infrastructure." https://www.wired.com/2016/04/the-critical-hole-at-the-heart-of-cell-phone-infrastructure/
[^31]: Crosby, S. (2009). "A Lesson In Timing Attacks." https://codahale.com/a-lesson-in-timing-attacks/
[^32]: Charles Proxy. "Web Debugging Proxy." https://www.charlesproxy.com/
[^33]: mitmproxy. "A free and open source interactive HTTPS proxy." https://mitmproxy.org/
[^34]: Mozilla. "Server Side TLS." https://wiki.mozilla.org/Security/Server_Side_TLS
[^35]: Auth0. "JWT.io." https://jwt.io/
[^36]: Auth0. (2015). "Critical vulnerabilities in JSON Web Token libraries." https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/
[^37]: CVE. "CVE-2015-9235." https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-9235
[^38]: Redis. "In-memory data structure store." https://redis.io/
[^39]: OWASP. "Cross-site Scripting (XSS)." https://owasp.org/www-community/attacks/xss/
[^40]: MDN. "SameSite cookies." https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
[^41]: Lodderstedt, T. et al. "OAuth 2.0 Security Best Current Practice." IETF Draft. https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics
[^42]: Jones, M., Bradley, J., and Sakimura, N. (2015). "JSON Web Signature (JWS)." RFC 7515, IETF. https://datatracker.ietf.org/doc/html/rfc7515
[^43]: Jones, M. and Hildebrand, J. (2015). "JSON Web Encryption (JWE)." RFC 7516, IETF. https://datatracker.ietf.org/doc/html/rfc7516
[^44]: Microsoft. "Azure Active Directory." https://azure.microsoft.com/en-us/services/active-directory/
[^45]: Keycloak. "Open Source Identity and Access Management." https://www.keycloak.org/
[^46]: Okta. "Identity and Access Management." https://www.okta.com/
[^47]: Auth0. "Identity Platform." https://auth0.com/
[^48]: HashiCorp. "Vault - Manage Secrets and Protect Sensitive Data." https://www.vaultproject.io/
[^49]: CyberArk. "Privileged Access Management." https://www.cyberark.com/
[^50]: AWS. "AWS Secrets Manager." https://aws.amazon.com/secrets-manager/
[^51]: Polli, R. and Martinez, A. "RateLimit Header Fields for HTTP." IETF Draft. https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/
[^52]: Nottingham, M. and Fielding, R. (2012). "Additional HTTP Status Codes." RFC 6585, IETF. https://datatracker.ietf.org/doc/html/rfc6585
[^53]: Fingerprint. "FingerprintJS." https://fingerprint.com/
[^54]: Tor Project. "Anonymity Online." https://www.torproject.org/
[^55]: European Union. "General Data Protection Regulation (GDPR)." https://gdpr.eu/
[^56]: OWASP. "Session Management Cheat Sheet." https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: Pragmatism](02-pragmatism.md) | [Next: Design Principles](04-design-principles.md)
