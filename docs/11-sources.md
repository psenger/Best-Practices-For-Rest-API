# Sources and Further Reading

[Index](../README.md) | [Previous: Design Extensibility](10-design-extensibility.md)

---

A curated reference list for the topics covered in this guide, organised by category.

## Standards and RFCs

- RFC 5861 (Stale-While-Revalidate): https://datatracker.ietf.org/doc/html/rfc5861
- RFC 6238 (TOTP): https://datatracker.ietf.org/doc/html/rfc6238
- RFC 6455 (WebSocket Protocol): https://datatracker.ietf.org/doc/html/rfc6455
- RFC 6585 (Additional HTTP Status Codes): https://datatracker.ietf.org/doc/html/rfc6585
- RFC 6797 (HSTS): https://datatracker.ietf.org/doc/html/rfc6797
- RFC 7232 (Conditional Requests): https://datatracker.ietf.org/doc/html/rfc7232
- RFC 7234 (HTTP Caching): https://datatracker.ietf.org/doc/html/rfc7234
- RFC 7515 (JWS — JSON Web Signature): https://datatracker.ietf.org/doc/html/rfc7515
- RFC 7516 (JWE — JSON Web Encryption): https://datatracker.ietf.org/doc/html/rfc7516
- RFC 7519 (JWT — JSON Web Token): https://datatracker.ietf.org/doc/html/rfc7519
- RFC 7636 (PKCE): https://datatracker.ietf.org/doc/html/rfc7636
- RFC 8288 (Web Linking): https://datatracker.ietf.org/doc/html/rfc8288
- RFC 8446 (TLS 1.3): https://datatracker.ietf.org/doc/html/rfc8446
- RFC 8594 (Sunset Header): https://datatracker.ietf.org/doc/html/rfc8594
- RFC 9110 (HTTP Semantics): https://httpwg.org/specs/rfc9110.html
- RFC 9449 (DPoP — Demonstration of Proof of Possession): https://datatracker.ietf.org/doc/html/rfc9449
- RFC 9457 (Problem Details for HTTP APIs): https://datatracker.ietf.org/doc/html/rfc9457
- RFC 9470 (OAuth 2.0 Step-Up Authentication): https://datatracker.ietf.org/doc/html/rfc9470
- RFC 9700 (OAuth 2.0 Security Best Current Practice): https://datatracker.ietf.org/doc/html/rfc9700
- IANA Link Relations: https://www.iana.org/assignments/link-relations/link-relations.xhtml
- W3C Trace Context: https://www.w3.org/TR/trace-context/
- W3C Content Security Policy Level 3: https://www.w3.org/TR/CSP3/
- WHATWG Server-Sent Events: https://html.spec.whatwg.org/multipage/server-sent-events.html
- IETF Rate Limit Headers (draft): https://datatracker.ietf.org/doc/draft-ietf-httpapi-ratelimit-headers/
- Semantic Versioning: https://semver.org/

---

## Security

- OWASP API Security Top 10 (2023): https://owasp.org/API-Security/
- OWASP REST Security Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- OWASP CSRF Prevention Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html
- OWASP Input Validation Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html
- OWASP Password Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- OWASP Session Management Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP Threat Modeling Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
- NIST Zero Trust Architecture (SP 800-207): https://csrc.nist.gov/publications/detail/sp/800-207/final
- NIST SP 800-63-4 (Digital Identity Guidelines): https://pages.nist.gov/800-63-4/
- NIST SP 800-63B (Authentication Assurance): https://pages.nist.gov/800-63-3/sp800-63b.html
- CISA Zero Trust Maturity Model v2.0: https://www.cisa.gov/zero-trust-maturity-model
- FIDO Alliance Passkeys: https://fidoalliance.org/passkeys/
- Fetch Metadata Headers: https://web.dev/articles/fetch-metadata
- MDN CORS: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
- Mozilla Observatory (security header scanner): https://observatory.mozilla.org
- ISO/IEC 27001 Information Security Management: https://www.iso.org/standard/27001
- SOC 2 Trust Services Criteria: https://www.aicpa.org/interestareas/frc/assuranceadvisoryservices/aaborgunits/soc2

---

## Auth and Identity

- OAuth 2.0: https://oauth.net/2/
- OAuth 2.1 (draft): https://oauth.net/2.1/
- OpenID Connect: https://openid.net/connect/
- JWT.io: https://jwt.io/
- Auth0. "Critical vulnerabilities in JSON Web Token libraries." https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/
- Keycloak (open source identity provider): https://www.keycloak.org/
- Okta: https://www.okta.com/
- Auth0: https://auth0.com/
- HashiCorp Vault: https://www.vaultproject.io/
- AWS Secrets Manager: https://aws.amazon.com/secrets-manager/

---

## Design and Architecture

- REST (Fielding's dissertation): https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
- Google API Design Guide: https://cloud.google.com/apis/design
- Zalando RESTful API Guidelines: https://opensource.zalando.com/restful-api-guidelines/
- API Stylebook: http://apistylebook.com/
- API Gateway Pattern: https://microservices.io/patterns/apigateway.html
- Microsoft API Gateway Pattern: https://learn.microsoft.com/en-us/azure/architecture/microservices/design/gateway
- Microsoft Bulkhead Pattern: https://learn.microsoft.com/en-us/azure/architecture/patterns/bulkhead
- BFF Pattern (Sam Newman): https://samnewman.io/patterns/architectural/bff/
- What is a service mesh (Linkerd): https://linkerd.io/what-is-a-service-mesh/
- Martin Fowler on Circuit Breaker: https://martinfowler.com/bliki/CircuitBreaker.html
- Martin Fowler — Two Hard Things (cache invalidation): https://martinfowler.com/bliki/TwoHardThings.html
- Hexagonal Architecture: https://alistair.cockburn.us/hexagonal-architecture/
- Hyrum's Law: https://www.hyrumslaw.com/
- Postel's Law (Robustness Principle) RFC 761: https://datatracker.ietf.org/doc/html/rfc761
- JSON:API Specification: https://jsonapi.org/
- Pact Contract Testing: https://pact.io/
- SOLID Principles (Martin): https://web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf

---

## Observability and Resilience

- Google SRE Book (Monitoring): https://sre.google/sre-book/monitoring-distributed-systems/
- Google SRE Book (Service Level Objectives): https://sre.google/sre-book/service-level-objectives/
- OpenTelemetry: https://opentelemetry.io/
- RED Method (Tom Wilkie): https://grafana.com/blog/2018/08/02/the-red-method-how-to-instrument-your-services/
- Prometheus Exposition Format: https://prometheus.io/docs/instrumenting/exposition_formats/
- AWS Exponential Backoff and Jitter: https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
- Netflix API Resilience: https://netflixtechblog.com/making-the-netflix-api-more-resilient-a8ec62f4b4f7
- Principles of Chaos Engineering: https://principlesofchaos.org/
- opossum (Circuit Breaker for Node.js): https://github.com/nodeshift/opossum
- resilience4j (Java): https://github.com/resilience4j/resilience4j
- Polly (.NET): https://github.com/App-vNext/Polly

---

## GraphQL and Real-Time

- GraphQL: https://graphql.org/
- GraphQL Specification: https://spec.graphql.org/
- GraphQL Subscriptions: https://graphql.org/blog/subscriptions-in-graphql-and-relay/
- DataLoader: https://github.com/graphql/dataloader
- GraphQL Query Complexity: https://github.com/slicknode/graphql-query-complexity
- Apollo Client: https://www.apollographql.com/docs/react/
- Apollo Server APQ: https://www.apollographql.com/docs/apollo-server/performance/apq/
- Relay: https://relay.dev/
- Relay Cursor Connections Spec: https://relay.dev/graphql/connections.htm
- Shopify — Solving N+1 in GraphQL: https://shopify.engineering/solving-the-n-1-problem-for-graphql-through-batching
- MDN WebSocket API: https://developer.mozilla.org/en-US/docs/Web/API/WebSocket
- MDN EventSource API: https://developer.mozilla.org/en-US/docs/Web/API/EventSource
- MDN Server-Sent Events guide: https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events

---

## Tooling and Libraries

- OpenAPI Specification: https://www.openapis.org/
- Swagger UI: https://swagger.io/tools/swagger-ui/
- Redoc: https://github.com/Redocly/redoc
- OpenAPI Generator: https://openapi-generator.tech/
- Kong: https://konghq.com/products/kong-gateway
- Redis: https://redis.io/
- FingerprintJS: https://fingerprint.com/

---

## API Design References (Real-World)

- Stripe API: https://stripe.com/docs/api
- Stripe Idempotent Requests: https://stripe.com/docs/api/idempotent_requests
- Stripe API Versioning: https://stripe.com/docs/api/versioning
- Stripe API Upgrades: https://stripe.com/docs/upgrades
- Stripe Metadata: https://docs.stripe.com/metadata
- Slack API Pagination: https://slack.engineering/evolving-api-pagination-at-slack/
- Slack Message Metadata: https://api.slack.com/metadata/using
- AWS Resource Tags: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html

---

## Pragmatism and Dependencies

- CNCF on avoiding lock-in: https://www.cncf.io/blog/2018/03/07/6-considerations-for-avoiding-lock-in-when-using-cloud-native-services/
- Thoughtworks Technology Radar: https://www.thoughtworks.com/radar
- Bundlephobia (npm bundle size checker): https://bundlephobia.com/
- Socket.dev (supply chain security): https://socket.dev/

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: Design Extensibility](10-design-extensibility.md)
