# Best Practices for Building REST APIs

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

I've been building APIs for Service Oriented Applications for two decades. Many API design opinions and guidelines I have found are either academic in nature or less real world or practical. My goal with this document is to describe my version of best practices for a pragmatic API approach based on my experience and as a framework for my thoughts. I've found the following items to be the key to the success of my systems: The Human Aspect, Security and Permissions, and Implementation.

> [!TIP]
> **Want an AI-powered review of your API design?** This knowledge base powers the
> [`review-api-design`](https://github.com/psenger/ai-agent-skills/tree/main/skills/review-api-design) skill — a
> **Skill 2.0** compatible with any LLM that supports the Skill 2.0 standard. Install it and get structured,
> severity-tagged API design reviews drawn from the 110+ standards, RFCs, and guidelines curated here.
>
> **Eval results — skill vs raw Claude:**
>
> | Scenario | With Skill | Raw Claude | Delta |
> |----------|-----------|------------|-------|
> | Minimal CRUD Endpoints | 100% | 50% | +50% |
> | OpenAPI Spec (Payments) | 100% | 62% | +38% |
> | Vague Verbal Description | 82% | 36% | +46% |
> | **Overall** | **94%** | **50%** | **+44%** |
>
> The skill catches what raw Claude misses: sequential ID enumeration, BOLA risks, missing health endpoints,
> rate limiting gaps, RFC-specific citations, and more — all while staying at the design level instead of drifting
> into implementation.
>
> **Install it now:**
> ```bash
> npx skills add psenger/ai-agent-skills --skill review-api-design
> ```

## Table of Contents

1. [The Human Aspect](docs/01-human-aspect.md)
   - Adoption, Ease of Use, Documentation, Stability, Assurance
   - Intuitive Design
   - Consistency and Documentation

2. [Pragmatism](docs/02-pragmatism.md)
   - Avoid Dependency Bloat
   - Framework Lock-in
   - Build vs Buy
   - AI-Assisted API Development

3. [Security and Permissions](docs/03-security.md)
   - Security Principles (Zero Trust, Defense in Depth, CIA Triad, Fail Secure)
   - Authentication vs Authorisation
   - Circles of Trust
   - Passwords and OTP
   - Tokens (Signature Verification, JTI, DPoP, BFF Pattern, Storage, Access/Refresh, JWS/JWE)
   - Risk-Based Assessment (RBA) and Step-Up Authentication
   - Perimeter vs Distributed Security
   - IAM, Identity, and PAM
   - Rate Limiting and Device Fingerprinting
   - Session Management and Multi-Factor Authentication
   - BOLA, Enumeration Prevention, Information Disclosure
   - CSRF, Security Headers, Security Logging
   - OWASP API Security Top 10 (2023)

4. [Design Principles](docs/04-design-principles.md)
   - Contract-First Development
   - Domain-Driven Design
   - API as a Product
   - Standards and Consistency (Naming, Versioning, CRUD)
   - Self Discovery and HATEOAS (with trade-offs)
   - Idempotency
   - Conditional Requests
   - CORS, Health Checks, Request Tracing

5. [Resilience](docs/05-resilience.md)
   - Service Unavailable
   - Exponential Backoff with Jitter
   - Circuit Breaker Pattern
   - Graceful Degradation
   - Timeouts and Bulkheads
   - Caching (Headers, Layers, Invalidation, Stampedes)
   - Observability (SLIs/SLOs/Error Budgets, RED Metrics, Structured Logging, Distributed Tracing, Alerting)

6. [Payloads and Errors](docs/06-payloads-and-errors.md)
   - Response Structure (Envelope vs No Envelope)
   - Field Selection (Projection)
   - Pagination (Offset, Cursor, Link-based, Header-based)
   - Errors (RFC 9457 Problem Details)
   - Identifiers (UUIDs, Type-prefixed IDs)
   - Content Negotiation

7. [API Communication Patterns](docs/07-api-communication-patterns.md)
   - REST, GraphQL, WebSockets, and Server-Sent Events (SSE)
   - Comparison Matrix and Decision Guide
   - Hybrid Architectures
   - GraphQL Performance (N+1, Query Complexity, Caching)
   - Anti-Patterns and Red Flags

8. [API Gateways](docs/08-api-gateways.md)
   - What API Gateways Do
   - Common Products (Apigee, Kong, AWS API Gateway, Azure APIM)
   - Gateway Patterns (Proxy, Transformation, Aggregation, Pipeline)
   - Pros and Cons
   - When to Use (and When to Skip)

9. [Glossary](docs/09-glossary.md)
   - Technical terms, acronyms, and concepts (mTLS, PKCE, JTI, HATEOAS, OAuth, JWT, and more)

10. [Design Extensibility and Evolution](docs/10-design-extensibility.md)
    - Fixed vs Variable Arity
    - Metadata as an Extension Point (Stripe, AWS, Slack patterns)
    - Response Evolution Strategies
    - SOLID Principles Applied to APIs
    - Postel's Law and Hyrum's Law

11. [Sources and Further Reading](docs/11-sources.md)
    - Standards and RFCs
    - Security, Auth, Design, Observability, GraphQL, Real-Time, Tooling

---

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a PR — all changes must start with a GitHub issue.

## License

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

You are free to share and adapt this material for any purpose, even commercially, as long as you give appropriate credit to the original author.
