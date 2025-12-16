# API Gateways

[Index](../README.md) | [Previous: GraphQL vs REST](07-graphql-vs-rest.md)

---

API gateways[^1] sit between clients and your backend services, providing a single entry point for API traffic. They handle cross-cutting concerns so your services don't have to. Once a DevOps told me "Api Gateways are anti-patterns"... An API Gateway acts as a reverse proxy and single entry point for clients to access multiple backend services and are a well established architectural pattern[^2].

## What API Gateways Do

At their core, API gateways handle:

- **Routing** - Direct requests to the appropriate backend service
- **Authentication** - Validate tokens, API keys, certificates
- **Rate limiting** - Protect backends from traffic spikes
- **Transformation** - Modify requests and responses (headers, body, format)
- **Aggregation** - Combine multiple backend calls into a single response
- **Caching** - Cache responses to reduce backend load
- **Monitoring** - Collect metrics, logs, and traces

```
                    ┌─────────────────────┐
   Clients ────────▶│    API Gateway      │
                    │                     │
                    │  • Auth             │
                    │  • Rate limiting    │
                    │  • Routing          │
                    │  • Transformation   │
                    └──────────┬──────────┘
                               │
           ┌───────────────────┼───────────────────┐
           ▼                   ▼                   ▼
    ┌────────────┐      ┌────────────┐      ┌────────────┐
    │  Service A │      │  Service B │      │  Service C │
    └────────────┘      └────────────┘      └────────────┘
```

## Common API Gateway Products

### Apigee (Google Cloud)

Enterprise-grade gateway[^3] with a focus on API lifecycle management.

**Strengths:**
- Comprehensive analytics and monetisation features
- Strong policy framework for complex transformations
- Developer portal for API documentation and key management
- Good for organisations selling APIs as products

**Weaknesses:**
- Complex to configure and operate
- Expensive, especially at scale
- Can become a bottleneck if over-customised
- Lock-in to Google Cloud ecosystem

### Kong

Open-source gateway[^4] built on NGINX[^5] with enterprise features available.

**Strengths:**
- Large plugin ecosystem
- Can run anywhere (Kubernetes, VMs, bare metal)
- Lower cost than enterprise alternatives
- Active community

**Weaknesses:**
- Enterprise features require paid subscription
- Plugin quality varies
- Configuration can be complex for advanced use cases

### AWS API Gateway

Managed service[^6] tightly integrated with AWS ecosystem.

**Strengths:**
- Serverless (no infrastructure to manage)
- Native integration with Lambda[^7], IAM[^8], Cognito[^9]
- Pay-per-request pricing at low volumes
- Automatic scaling

**Weaknesses:**
- Tight AWS coupling
- Limited transformation capabilities compared to Apigee
- Cold start latency with Lambda integration
- Can get expensive at high volumes

### Azure API Management

Microsoft's managed API gateway service[^10].

**Strengths:**
- Native Azure integration
- Good developer portal
- Policy engine for transformations
- Built-in caching

**Weaknesses:**
- Azure lock-in
- Complex pricing model
- Can be slow to deploy changes

### Other Options

- **Traefik**[^11] - Modern, cloud-native, good for Kubernetes
- **NGINX Plus**[^12] - High performance, familiar to ops teams
- **Envoy**[^13] - Service mesh focused, used by Istio[^14]
- **Tyk**[^15] - Open source with good Kubernetes support

## Gateway Patterns

### Simple Proxy

The gateway acts as a pass-through, adding cross-cutting concerns without modifying requests.

```
Client ──▶ Gateway (auth, rate limit) ──▶ Service
```

Use for: Basic protection of existing APIs with minimal changes.

### Request/Response Transformation

The gateway modifies requests before sending to backends and responses before returning to clients.

```
Client: { "user_name": "alice" }
         │
         ▼
Gateway: transforms to { "userName": "alice" }
         │
         ▼
Service: expects camelCase
```

Use for: Supporting legacy clients, format normalisation, header injection.

### Aggregation (Backend for Frontend)

The Backend for Frontend (BFF) pattern[^16] uses the gateway to combine multiple backend calls into a single response, reducing client round trips.

```
Client ──▶ Gateway ──┬──▶ User Service ────────┐
                     │                         │
                     ├──▶ Order Service ───────┤
                     │                         ▼
                     └──▶ Product Service ──▶ Gateway aggregates ──▶ Client
```

**Benefits:**
- Reduced latency (parallel backend calls)
- Simpler client code
- Hide internal service boundaries

**Risks:**
- Gateway becomes complex
- Failures are harder to handle (partial success?)
- Can mask performance problems in backends

Use for: Mobile clients with bandwidth constraints, single-page applications needing multiple data sources.

### Pipeline/Mediation

The gateway applies a sequence of transformations or validations in order.

```
Request ──▶ Validate ──▶ Enrich ──▶ Transform ──▶ Route ──▶ Backend
                │           │           │
                │           │           └── Change format
                │           └── Add user context from cache
                └── Schema validation
```

This is common in Apigee and Azure APIM where you define "policies" that execute in sequence.

**Benefits:**
- Clear separation of concerns
- Reusable policy components
- Declarative configuration

**Risks:**
- Hard to debug when pipelines get long
- Performance impact from many transformations
- Logic buried in gateway config

### Service Mesh Integration

For microservices, the gateway handles north-south traffic (external to internal) while a service mesh[^17] handles east-west traffic (service to service).

```
External                    Internal
   │                           │
   ▼                           │
┌──────────┐                   │
│ Gateway  │                   │
└────┬─────┘                   │
     │                         │
     ▼                         ▼
┌─────────────────────────────────┐
│        Service Mesh             │
│  ┌─────┐  ┌─────┐  ┌─────┐      │
│  │ Svc │──│ Svc │──│ Svc │      │
│  └─────┘  └─────┘  └─────┘      │
└─────────────────────────────────┘
```

The gateway handles external concerns (API versioning, public rate limits), while the mesh handles internal concerns (mTLS[^18], circuit breaking[^19], retries).

## Pros and Cons of API Gateways

### Advantages

**Centralised cross-cutting concerns.** Authentication, rate limiting, and logging are implemented once, not in every service.

**Single entry point.** Simplifies client configuration and enables consistent API versioning.

**Backend flexibility.** You can refactor, split, or migrate services without changing client contracts.

**Security boundary.** External traffic never touches backends directly.

**Observability.** Single point to collect metrics, logs, and traces for all API traffic.

### Disadvantages

**Single point of failure.** If the gateway goes down, everything goes down. Requires high availability configuration.

**Added latency.** Every request goes through an extra hop. Usually milliseconds, but it adds up.

**Operational complexity.** Another system to monitor, secure, update, and scale.

**Logic sprawl.** The temptation to put "just one more thing" in the gateway leads to unmaintainable configurations.

**Vendor lock-in.** Gateway-specific policies and transformations don't port easily.

**Cost.** Enterprise gateways are expensive. Even managed services add up at scale.

## When to Use an API Gateway

**Use a gateway when:**
- You have multiple backend services that need consistent authentication
- External clients need a stable API while internals evolve
- You need to aggregate or transform responses
- Rate limiting and throttling are critical
- You want a developer portal for API documentation

**Skip the gateway when:**
- You have a single service (just put these concerns in the service)
- Internal-only APIs where services call each other directly
- You're adding complexity without clear benefit
- Your team can't operate another critical system

## Anti-Patterns

### Gateway as Business Logic Layer

If your gateway contains significant business logic (calculations, conditional flows, data lookups), you've built a monolith. Keep business logic in services.

### Over-Aggregation

Aggregating 10+ backend calls in the gateway creates a fragile, hard-to-debug system. Consider whether the client really needs all that data, or if you need a dedicated BFF service.

### Configuration Drift

When gateway configuration is edited manually in production without version control, you lose reproducibility. Treat gateway config as code.

### Ignoring Failures

Gateway aggregation must handle partial failures gracefully. If one of five backends fails, what does the client get? Define this explicitly.

## Practical Recommendations

1. **Start simple.** Begin with basic routing and auth. Add complexity only when needed.

2. **Version your configuration.** Store gateway config in git. Review changes like code.

3. **Monitor gateway performance.** Track latency added by the gateway itself.

4. **Plan for failure.** What happens when the gateway is unavailable? Have a runbook.

5. **Limit transformation complexity.** If transformations get complex, build a dedicated service instead.

6. **Document policies.** Future maintainers need to understand why each policy exists.

---

## References

[^1]: Richardson, Chris. "Pattern: API Gateway / Backends for Frontends." Microservices.io. https://microservices.io/patterns/apigateway.html
[^2]: Microsoft. "API gateway pattern." Azure Architecture Center. https://learn.microsoft.com/en-us/azure/architecture/microservices/design/gateway
[^3]: Google Cloud. "Apigee API Management." https://cloud.google.com/apigee
[^4]: Kong Inc. "Kong Gateway." https://konghq.com/products/kong-gateway
[^5]: NGINX. "NGINX High Performance Load Balancer, Web Server, & Reverse Proxy." https://www.nginx.com/
[^6]: Amazon Web Services. "Amazon API Gateway." https://aws.amazon.com/api-gateway/
[^7]: Amazon Web Services. "AWS Lambda - Serverless Compute." https://aws.amazon.com/lambda/
[^8]: Amazon Web Services. "AWS Identity and Access Management (IAM)." https://aws.amazon.com/iam/
[^9]: Amazon Web Services. "Amazon Cognito." https://aws.amazon.com/cognito/
[^10]: Microsoft. "Azure API Management." https://azure.microsoft.com/en-us/products/api-management
[^11]: Traefik Labs. "Traefik - The Cloud Native Application Proxy." https://traefik.io/traefik/
[^12]: NGINX. "NGINX Plus." https://www.nginx.com/products/nginx/
[^13]: Envoy Proxy. "Envoy is an open source edge and service proxy." https://www.envoyproxy.io/
[^14]: Istio. "Istio Service Mesh." https://istio.io/
[^15]: Tyk. "Tyk API Gateway." https://tyk.io/
[^16]: Newman, Sam. (2015). "Building Microservices." O'Reilly Media. Chapter 4: Integration - Backend for Frontend Pattern. https://www.oreilly.com/library/view/building-microservices/9781491950340/
[^17]: Linkerd. "What is a service mesh?" https://linkerd.io/what-is-a-service-mesh/
[^18]: Cloudflare. "What is mTLS?" https://www.cloudflare.com/learning/access-management/what-is-mutual-tls/
[^19]: Fowler, Martin. (2014). "CircuitBreaker." https://martinfowler.com/bliki/CircuitBreaker.html

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: GraphQL vs REST](07-graphql-vs-rest.md) | [Next: Glossary](09-glossary.md)
