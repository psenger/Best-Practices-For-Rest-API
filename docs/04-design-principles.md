# Design Principles

[Index](../README.md) | [Previous: Security and Permissions](03-security.md)

---

## Contract-First Development

Define your API contract before writing implementation code. This isn't bureaucracy; it's pragmatism.

**Why contract-first:**

- **Parallel development** - Frontend and backend teams can work simultaneously. Frontend mocks the API while backend implements it.
- **Catch design flaws early** - It's cheaper to change a spec than refactor code.
- **Better APIs** - When you design without implementation pressure, you make better decisions.
- **Documentation is always current** - The spec is the source of truth, not an afterthought.

**How to do it:**

1. Write the OpenAPI[^1] spec first
2. Review with consumers (frontend, mobile, partners)
3. Generate server stubs and client SDKs using OpenAPI Generator[^2]
4. Implement against the contract
5. Use contract testing[^3] to ensure implementation matches spec

```yaml
# Design the contract first
openapi: 3.0.3
info:
  title: Orders API
  version: 1.0.0
paths:
  /orders:
    post:
      summary: Create an order
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
      responses:
        '201':
          description: Order created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
```

The alternative, code-first with generated docs, leads to APIs shaped by implementation convenience rather than consumer needs.

## Domain-Driven Design

Your API should reflect your business domain, not your database schema or internal architecture. See Eric Evans' Domain-Driven Design[^4] and Martin Fowler's DDD summary[^5].

**Ubiquitous language**[^6] - Use the same terms your business uses. If the business calls it an "order," don't call it a "purchase transaction" in your API. If they say "cancel," don't say "terminate."

**Bounded contexts**[^7] - Large systems have multiple domains. An "order" in shipping means something different than an "order" in billing. Don't force a single unified model; let each context define its own.

**Aggregates**[^8] - Group related entities that change together. An order and its line items are an aggregate. Expose the aggregate root (`/orders/{id}`) not the internals (`/line-items/{id}`).

```
# Good - reflects domain concepts
POST /orders
POST /orders/{id}/cancel
GET  /customers/{id}/orders

# Bad - exposes implementation
POST /order_records
PUT  /orders/{id}/status  (with body: { "status": "cancelled" })
GET  /database/customers/join/orders
```

**Anti-corruption layer**[^9] - When integrating with legacy systems or third-party APIs, don't let their models pollute your domain. Translate at the boundary.

## API as a Product

Treat your API as a product, not a by-product. Someone will consume it, and their experience matters. See API as a Product[^10] for more on this mindset.

- **Consistency is kindness** - Inconsistent APIs waste developers' time. If one endpoint uses `created_at` and another uses `createdDate`, someone will get it wrong.
- **Versioning is a promise** - When you publish v1, you're promising not to break it. Take that seriously.
- **Errors are documentation** - A good error message teaches the developer what went wrong and how to fix it.
- **Deprecation is communication** - Don't just remove things. Warn, provide migration paths, give timelines.

## Standards and Consistency

Keep your services designed to serve **resources**. Otherwise you risk your services becoming remote procedure calls. REST[^11] is Representational State Transfer, not RPC.

### Naming Convention

The naming convention is very important because it implies consistency. The naming convention should not leak implementation details. It should relate to resources. See Google's API Design Guide[^12] for comprehensive naming guidance.

#### Nouns

Endpoints should be nouns, such as **books** or **users**. Names that are verbs or adjectives are problematic:

| Bad | Good |
|-----|------|
| `/doPayroll` | `/payroll` |
| `/createUser` | `POST /users` |
| `/getUserById` | `GET /users/{id}` |

#### Versioning

Resources should be versioned. There are two main approaches:

**URL-based versioning** works best for API teams:

```
/v2/books
```

This makes it easy to stand up separate servers behind a load balancer without convoluting code with versioning concerns. Use only the major version number. Avoid `/v2.14.2/books` which becomes unmanageable.

**Header-based versioning** offers more flexibility:

```
Accept-Version: 2
```

This allows clients to upgrade independently but adds complexity to routing.

Refer to Semantic Versioning[^13] for version numbering. Use Sunset headers per RFC 8594[^14] to communicate deprecation timelines.

When I found that I needed to support multiple versions, I usually added statistics on the endpoint to find when it was no longer being used, and removed it only after inactivity.

#### Plural

Resources should always have plural names:

```
GET /books
GET /books/1234
POST /books
```

Avoid singular names like `/book`. The plural form makes it clear that the endpoint represents a collection, and individual resources are accessed by ID.

#### CRUD

Create, Read, Update, Delete should be represented through HTTP methods per RFC 9110[^15]:

| Operation | HTTP Verb | Example |
|-----------|-----------|---------|
| Create | POST | `POST /books` |
| Read (list) | GET | `GET /books` |
| Read (single) | GET | `GET /books/123` |
| Update (full) | PUT | `PUT /books/123` |
| Update (partial) | PATCH | `PATCH /books/123` |
| Delete | DELETE | `DELETE /books/123` |

## Self Discovery and HATEOAS

Self-discovery, or HATEOAS[^16] (Hypermedia as the Engine of Application State), means that links within the response enable discovery of related endpoints and actions. The idea is powerful: clients navigate the API by following links rather than constructing URLs, reducing coupling between client and server. See Roy Fielding's explanation[^17].

When including links, always include a **rel** value to describe the relationship. Use IANA link relations[^18] where applicable:

```json
{
  "id": "book_abc123",
  "title": "API Design Patterns",
  "links": [
    { "rel": "self", "href": "/books/book_abc123" },
    { "rel": "author", "href": "/authors/auth_xyz789" },
    { "rel": "reviews", "href": "/books/book_abc123/reviews" }
  ]
}
```

Document all `rel` types in your API specification. This allows consumer developers to create global handlers rather than one-off designs for each endpoint.

### The Reality of HATEOAS

HATEOAS is academically elegant but rarely implemented fully in practice. Here's why:

**Implementation cost is high.** Every response needs link generation logic. Links must be context-aware (different users see different available actions). Maintaining this across a large API is substantial work.

**Clients rarely use it.** Most API consumers want fit-for-purpose endpoints that return exactly what they need for a specific use case. They don't want to navigate a web of links to assemble data. Frontend developers typically hardcode URLs because it's simpler and more predictable.

**Entity-centric vs use-case-centric.** HATEOAS pushes you toward microscopic entity relationships: a book links to its author, the author links to their other books, those books link to their reviews. But real applications need "get me everything for the book detail page" not "let me traverse the entity graph." This is where REST's resource model and GraphQL's query model diverge.

**State explosion.** If links represent available actions based on state (an order can be cancelled only when it's pending), link generation becomes complex business logic. You're encoding state machines into every response.

### When HATEOAS makes sense

- **Public APIs with long-lived clients.** If you can't update clients when URLs change, link discovery helps.
- **Pagination.** Including `next`, `prev`, and `last` links is genuinely useful.
- **Discoverable APIs.** Developer portals and API explorers benefit from self-describing responses.
- **Workflow APIs.** When the API genuinely models a state machine (order processing, approval workflows), links showing available transitions add value.

### Pragmatic approach

Adopt HATEOAS selectively:

- Always include `self` links for resources
- Include pagination links for collections
- Include links for common related resources (author on a book)
- Skip the ideology of "clients should never construct URLs"

Most successful APIs use partial HATEOAS: links where they're useful, not religious adherence to the constraint.

## Idempotency

Idempotent operations produce the same result regardless of how many times they're called. GET, PUT, and DELETE are naturally idempotent. POST is not. See RFC 9110 on idempotency[^19].

For non-idempotent operations, use an **idempotency key**. Stripe's implementation[^20] is a good reference:

```http
POST /payments
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "amount": 1000,
  "currency": "USD"
}
```

The server should:

1. Check if this key has been seen before
2. If yes, return the cached response
3. If no, process the request and cache the response

This prevents duplicate operations from network retries or client bugs. Keys should expire after a reasonable period (24-48 hours).

## Conditional Requests

Use ETags per RFC 7232[^21] and conditional headers to optimise caching and prevent update conflicts.

**For caching (GET requests):**

```http
GET /books/123
If-None-Match: "abc123"
```

Returns `304 Not Modified` if unchanged, saving bandwidth.

**For optimistic locking (PUT/PATCH requests):**

```http
PUT /books/123
If-Match: "abc123"
Content-Type: application/json

{ "title": "Updated Title" }
```

Returns `412 Precondition Failed` if the resource has changed since the client last fetched it, preventing lost updates. See MDN's guide on conditional requests[^22].

## CORS

Cross-Origin Resource Sharing (CORS)[^23] is essential for browser-based API consumers. Configure these headers:

```http
Access-Control-Allow-Origin: https://trusted-domain.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization, Idempotency-Key
Access-Control-Max-Age: 86400
```

**Guidelines:**

- Never use `Access-Control-Allow-Origin: *` with credentials
- Validate the `Origin` header against an allowlist
- Keep preflight cache (`Max-Age`) high for performance
- Only expose headers clients actually need

See MDN's CORS guide[^24] for detailed implementation guidance.

## Health Checks

Provide health check endpoints for load balancers and orchestrators. See Kubernetes health check patterns[^25].

**Liveness** - Is the service running?

```
GET /health/live
200 OK
```

**Readiness** - Is the service ready to accept traffic?

```http
GET /health/ready

{
  "status": "healthy",
  "checks": {
    "database": "up",
    "cache": "up",
    "external_api": "degraded"
  }
}
```

Return `200` for healthy, `503` for unhealthy. Keep liveness checks simple (no dependencies). Readiness checks can verify database connections and critical dependencies.

## Request Tracing

Include correlation IDs for distributed tracing. See W3C Trace Context[^26] and OpenTelemetry[^27].

```http
GET /orders/123
X-Request-ID: req_abc123
X-Correlation-ID: corr_xyz789
```

- **Request ID** - Unique to this request, generated by the API if not provided
- **Correlation ID** - Passed through the entire call chain across services

Log these IDs in every service. This makes debugging distributed systems tractable.

## Parameters

URLs are visible in logs, browser history, and network monitoring tools. Never expose sensitive information in URL parameters or path segments. See OWASP's guidance on sensitive data[^28].

**Path parameters** - Use for resource identification:

```
GET /users/123
GET /users/123/orders/456
```

**Query parameters** - Use for filtering, sorting, and pagination:

```
GET /users?status=active&sort=-createdAt&limit=20
```

**Request body** - Use for sensitive data or complex payloads:

```http
POST /auth/login
Content-Type: application/json

{ "email": "user@example.com", "password": "..." }
```

If an entity has a unique ID, use only that ID in the path. Don't include redundant parent IDs even for nested resources unless required for authorization checks.

## Attribute Convention

Choose a naming convention and apply it consistently. The most common options:

| Convention | Example | Common in |
|------------|---------|-----------|
| camelCase | `firstName`, `createdAt` | JavaScript, Java |
| snake_case | `first_name`, `created_at` | Python, Ruby, databases |
| kebab-case | `first-name`, `created-at` | URLs, CSS (avoid in JSON) |

**camelCase** is widely adopted because:

- JavaScript (the dominant API consumer language) uses it natively
- JSON[^29] originated from JavaScript
- Most API documentation examples use it

Whatever you choose, be consistent across all endpoints and responses. Don't mix conventions within the same API. See Google's JSON style guide[^30].

---

## References

[^1]: OpenAPI Initiative. "OpenAPI Specification." https://www.openapis.org/
[^2]: OpenAPI Generator. "Generate clients, servers, and documentation from OpenAPI specifications." https://openapi-generator.tech/
[^3]: Pact Foundation. "Pact - Contract Testing." https://pact.io/
[^4]: Evans, Eric. (2003). "Domain-Driven Design: Tackling Complexity in the Heart of Software." Addison-Wesley. https://www.domainlanguage.com/ddd/
[^5]: Fowler, Martin. "Domain-Driven Design." https://martinfowler.com/bliki/DomainDrivenDesign.html
[^6]: Fowler, Martin. "Ubiquitous Language." https://martinfowler.com/bliki/UbiquitousLanguage.html
[^7]: Fowler, Martin. "Bounded Context." https://martinfowler.com/bliki/BoundedContext.html
[^8]: Fowler, Martin. "DDD Aggregate." https://martinfowler.com/bliki/DDD_Aggregate.html
[^9]: Microsoft. "Anti-Corruption Layer pattern." Azure Architecture Patterns. https://docs.microsoft.com/en-us/azure/architecture/patterns/anti-corruption-layer
[^10]: APIScene. "API as a Product." https://www.apiscene.io/api-as-a-product/
[^11]: Fielding, Roy Thomas. (2000). "Architectural Styles and the Design of Network-based Software Architectures." Doctoral dissertation, University of California, Irvine. Chapter 5: REST. https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
[^12]: Google Cloud. "API Design Guide." https://cloud.google.com/apis/design
[^13]: Preston-Werner, Tom. "Semantic Versioning 2.0.0." https://semver.org/
[^14]: Wilde, E. (2019). "The Sunset HTTP Header Field." RFC 8594, IETF. https://datatracker.ietf.org/doc/html/rfc8594
[^15]: Fielding, R. et al. (2022). "HTTP Semantics." RFC 9110, IETF. Section 9: Methods. https://httpwg.org/specs/rfc9110.html#methods
[^16]: Wikipedia. "HATEOAS." https://en.wikipedia.org/wiki/HATEOAS
[^17]: Fielding, Roy. (2008). "REST APIs must be hypertext-driven." https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven
[^18]: IANA. "Link Relations." https://www.iana.org/assignments/link-relations/link-relations.xhtml
[^19]: Fielding, R. et al. (2022). "HTTP Semantics." RFC 9110, IETF. Section 9.2.2: Idempotent Methods. https://httpwg.org/specs/rfc9110.html#idempotent.methods
[^20]: Stripe. "Idempotent Requests." Stripe API Documentation. https://stripe.com/docs/api/idempotent_requests
[^21]: Fielding, R. and Reschke, J. (2014). "Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests." RFC 7232, IETF. https://datatracker.ietf.org/doc/html/rfc7232
[^22]: MDN Web Docs. "HTTP conditional requests." https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests
[^23]: WHATWG. "Fetch Standard - CORS Protocol." https://fetch.spec.whatwg.org/#http-cors-protocol
[^24]: MDN Web Docs. "Cross-Origin Resource Sharing (CORS)." https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[^25]: Kubernetes. "Configure Liveness, Readiness and Startup Probes." https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
[^26]: W3C. "Trace Context." https://www.w3.org/TR/trace-context/
[^27]: OpenTelemetry. "High-quality, ubiquitous, and portable telemetry." https://opentelemetry.io/
[^28]: OWASP. "Sensitive Data Exposure." https://owasp.org/www-project-web-security-testing-guide/
[^29]: Crockford, Douglas. "Introducing JSON." https://www.json.org/
[^30]: Google. "Google JSON Style Guide." https://google.github.io/styleguide/jsoncstyleguide.xml

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: Security and Permissions](03-security.md) | [Next: Resilience](05-resilience.md)
