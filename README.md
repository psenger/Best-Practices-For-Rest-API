# Best Practices for Building REST APIs

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC_BY_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

I've been building APIs for Service Oriented Applications for two decades. Many API design opinions and guidelines I have found are either academic in nature or less real world or practical. My goal with this document is to describe my version of best practices for a pragmatic API approach based on my experience and as a framework for my thoughts. I've found the following items to be the key to the success of my systems: The Human Aspect, Security and Permissions, and Implementation.

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
   - Tokens (Signature Verification, JTI, Storage, Access/Refresh, JWS/JWE)
   - Risk-Based Assessment (RBA)
   - Perimeter vs Distributed Security
   - IAM, Identity, and PAM
   - Rate Limiting and Device Fingerprinting
   - Session Management

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

6. [Payloads and Errors](docs/06-payloads-and-errors.md)
   - Response Structure (Envelope vs No Envelope)
   - Field Selection (Projection)
   - Pagination (Offset, Cursor, Link-based, Header-based)
   - Errors (RFC 9457 Problem Details)
   - Identifiers (UUIDs, Type-prefixed IDs)
   - Content Negotiation

7. [GraphQL vs REST](docs/07-graphql-vs-rest.md)
   - Analysis of both approaches
   - GraphQL Performance Issues (N+1, Query Complexity, Caching)
   - When to Choose What
   - Common Anti-Patterns

8. [API Gateways](docs/08-api-gateways.md)
   - What API Gateways Do
   - Common Products (Apigee, Kong, AWS API Gateway, Azure APIM)
   - Gateway Patterns (Proxy, Transformation, Aggregation, Pipeline)
   - Pros and Cons
   - When to Use (and When to Skip)

9. [Glossary](docs/09-glossary.md)
   - Technical terms, acronyms, and concepts (mTLS, PKCE, JTI, HATEOAS, OAuth, JWT, and more)

---

## License

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

You are free to share and adapt this material for any purpose, even commercially, as long as you give appropriate credit to the original author.
