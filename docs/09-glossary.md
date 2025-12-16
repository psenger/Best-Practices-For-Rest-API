# Glossary

[Index](../README.md) | [Previous: API Gateways](08-api-gateways.md)

---

A reference guide for technical terms, acronyms, and concepts used throughout this documentation.

## A

### APM (Application Performance Monitoring)
Tools and practices for monitoring and managing the performance of software applications. APM solutions track metrics like response times, error rates, and throughput to identify bottlenecks and issues.

### Argon2
A password hashing algorithm that won the Password Hashing Competition in 2015. Argon2id is the recommended variant, combining resistance to both GPU-based and side-channel attacks.

## B

### bcrypt
A password hashing function based on the Blowfish cipher. It incorporates a salt and is intentionally slow to resist brute-force attacks. Widely used and battle-tested.

### BFF (Backend for Frontend)
An architectural pattern where a dedicated backend service is created for each frontend application (web, mobile, etc.). The BFF aggregates data from multiple services and formats it specifically for its client.

### Bulkhead
A resilience pattern that isolates failures to prevent them from spreading across a system. Named after ship compartments that contain flooding.

## C

### CDN (Content Delivery Network)
A geographically distributed network of servers that cache and deliver content to users from locations closer to them, reducing latency and improving performance.

### CIA Triad
The three pillars of information security: Confidentiality (data accessible only to authorised parties), Integrity (data is accurate and unaltered), and Availability (systems are accessible when needed).

### Circuit Breaker
A resilience pattern that prevents cascading failures by failing fast when a downstream service is unhealthy. Has three states: Closed (normal), Open (failing fast), and Half-Open (testing recovery).

### CORS (Cross-Origin Resource Sharing)
A security mechanism that allows web applications to make requests to domains other than their own. Controlled via HTTP headers like `Access-Control-Allow-Origin`.

### CRUD
The four basic operations of persistent storage: Create, Read, Update, and Delete. In REST APIs, these map to HTTP methods POST, GET, PUT/PATCH, and DELETE respectively.

### CSRF (Cross-Site Request Forgery)
An attack that tricks a user's browser into making unwanted requests to a site where they're authenticated. Mitigated with tokens or SameSite cookies.

## D

### DataLoader
A utility library for batching and caching data fetching operations, commonly used in GraphQL to solve the N+1 query problem.

### DDD (Domain-Driven Design)
A software design approach that focuses on modelling software to match the business domain. Key concepts include ubiquitous language, bounded contexts, and aggregates.

### DDoS (Distributed Denial of Service)
An attack that attempts to overwhelm a service with traffic from multiple sources, making it unavailable to legitimate users.

### Defense in Depth
A security strategy that layers multiple controls so that if one fails, others still protect the system.

### DoS (Denial of Service)
An attack that attempts to make a service unavailable by overwhelming it with requests or exploiting vulnerabilities.

## E

### ETag (Entity Tag)
An HTTP header that provides a version identifier for a resource. Used for cache validation and optimistic locking via `If-None-Match` and `If-Match` headers.

### Exponential Backoff
A retry strategy where the wait time between retries increases exponentially (e.g., 1s, 2s, 4s, 8s). Often combined with jitter to prevent thundering herd problems.

## G

### GraphQL
A query language for APIs developed by Facebook that allows clients to request exactly the data they need. Provides a single endpoint with a strongly typed schema.

## H

### HATEOAS (Hypermedia as the Engine of Application State)
A REST constraint where responses include links to related resources and available actions, enabling clients to navigate the API through hypermedia rather than hardcoded URLs.

### HOTP (HMAC-Based One-Time Password)
A one-time password algorithm that generates codes based on a counter value. Each code is valid until used. Defined in RFC 4226.

### HTTP-only Cookie
A cookie with the `HttpOnly` flag set, preventing JavaScript from accessing it. Used to protect session tokens from XSS attacks.

## I

### IAM (Identity and Access Management)
Systems and processes for managing digital identities and controlling access to resources. Encompasses authentication, authorisation, and user lifecycle management.

### Idempotency
The property of an operation where performing it multiple times produces the same result as performing it once. Critical for safe retry behaviour in distributed systems.

### Idempotency Key
A unique identifier sent with non-idempotent requests (like POST) to ensure the operation is only performed once, even if the request is retried.

## J

### Jitter
Random variation added to timing (e.g., retry delays) to prevent synchronised behaviour that could cause thundering herd problems.

### JTI (JWT ID)
A unique identifier claim within a JWT. Used to prevent replay attacks, enable token revocation, and bind tokens to specific devices or sessions.

### JWE (JSON Web Encryption)
A standard for encrypting JSON-based data structures. Unlike JWS, the payload is encrypted and cannot be read without the decryption key. Defined in RFC 7516.

### JWS (JSON Web Signature)
A standard for digitally signing JSON-based data structures. The payload is Base64-encoded and readable, but the signature ensures integrity. Defined in RFC 7515.

### JWT (JSON Web Token)
A compact, URL-safe token format for securely transmitting claims between parties. Consists of a header, payload, and signature. Defined in RFC 7519.

## M

### MFA (Multi-Factor Authentication)
Authentication requiring two or more verification factors: something you know (password), something you have (device), or something you are (biometrics).

### mTLS (Mutual TLS)
A security protocol where both client and server authenticate each other using certificates, rather than just the server authenticating to the client. Common in service-to-service communication.

## N

### N+1 Problem
A performance anti-pattern where fetching a collection of N items results in N additional queries to fetch related data. Common in GraphQL and ORM implementations. Solved with batching (e.g., DataLoader) or query optimisation.

## O

### OAuth 2.0
An authorisation framework that enables applications to obtain limited access to user accounts. Note: OAuth is for authorisation (what can you access), not authentication (who are you).

### OIDC (OpenID Connect)
An identity layer built on top of OAuth 2.0 that adds authentication. Provides a standardised way to verify user identity and obtain basic profile information.

### OpenAPI
A specification for describing REST APIs in a machine-readable format (YAML or JSON). Enables code generation, documentation, and contract testing. Formerly known as Swagger.

### OTP (One-Time Password)
A password valid for only one authentication session or transaction. See also TOTP and HOTP.

## P

### PAM (Privileged Access Management)
Controls and audits access to sensitive systems and data. Includes credential vaulting, just-in-time access, and session recording.

### PKCE (Proof Key for Code Exchange)
An OAuth 2.0 extension that prevents authorisation code interception attacks. Essential for mobile apps and single-page applications. Pronounced "pixy". Defined in RFC 7636.

### PoLP (Principle of Least Privilege)
A security principle stating that users and systems should only have the minimum permissions necessary to perform their tasks.

## R

### Rate Limiting
Controlling the number of requests a client can make within a time period. Protects against abuse and ensures fair resource allocation.

### RBA (Risk-Based Assessment)
An adaptive security approach that evaluates context (device, location, behaviour) to determine the appropriate level of authentication required.

### RBAC (Role-Based Access Control)
An access control model where permissions are assigned to roles, and users are assigned to roles. Simplifies permission management compared to per-user assignments.

### REST (Representational State Transfer)
An architectural style for distributed systems that uses HTTP methods and URLs to represent resources. Key constraints include statelessness, cacheability, and uniform interface.

### RFC (Request for Comments)
Documents published by the IETF that define internet standards, protocols, and best practices. Examples: RFC 9110 (HTTP), RFC 7519 (JWT).

## S

### SAML (Security Assertion Markup Language)
An XML-based standard for exchanging authentication and authorisation data between parties, commonly used for enterprise single sign-on (SSO).

### scrypt
A password hashing function designed to be memory-hard, making it expensive to attack with specialised hardware.

### Service Mesh
Infrastructure layer for handling service-to-service communication, typically handling concerns like mTLS, load balancing, and observability. Examples: Istio, Linkerd.

### SPA (Single Page Application)
A web application that loads a single HTML page and dynamically updates content without full page reloads. Common frameworks: React, Vue, Angular.

### SSO (Single Sign-On)
An authentication scheme that allows users to log in once and access multiple applications without re-authenticating.

### Step-Up Authentication
Requiring additional authentication (e.g., MFA) for sensitive operations, even when the user is already authenticated.

## T

### Thundering Herd
A problem where many clients simultaneously retry or reconnect after a failure, overwhelming the recovering service. Mitigated with jitter and backoff.

### TLS (Transport Layer Security)
A cryptographic protocol for secure communication over networks. TLS 1.3 is the current version. Successor to SSL.

### TOTP (Time-Based One-Time Password)
A one-time password algorithm that generates codes based on the current time and a shared secret. Codes are typically valid for 30 seconds. Defined in RFC 6238. Used by authenticator apps like Google Authenticator.

### TTL (Time to Live)
The duration for which data should be considered valid. Used in caching, DNS, and token expiration.

## U

### UUID (Universally Unique Identifier)
A 128-bit identifier designed to be unique across space and time. Example: `550e8400-e29b-41d4-a716-446655440000`. Defined in RFC 4122.

## W

### WAF (Web Application Firewall)
A security layer that filters and monitors HTTP traffic between a web application and the internet, protecting against common attacks like SQL injection and XSS.

## X

### XSS (Cross-Site Scripting)
A security vulnerability where attackers inject malicious scripts into web pages viewed by other users. Mitigated with input validation, output encoding, and Content Security Policy.

## Z

### Zero Trust
A security model that assumes no implicit trust based on network location. Every request must be authenticated and authorised, regardless of origin. "Never trust, always verify."

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: API Gateways](08-api-gateways.md) | [Index](../README.md)
