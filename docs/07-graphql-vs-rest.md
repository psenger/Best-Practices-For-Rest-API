# GraphQL vs REST APIs

[Index](../README.md) | [Previous: Payloads and Errors](06-payloads-and-errors.md)

---

## Context

When designing APIs, teams often debate between REST[^1] and GraphQL[^2]. Both are valid approaches, but they solve different problems and come with different trade-offs. This document captures the considerations and common pitfalls, particularly around GraphQL's performance characteristics.

## Decision Drivers

- Client flexibility requirements
- Network efficiency
- Caching requirements
- Team expertise
- Performance at scale
- Tooling maturity

## Considered Options

1. REST (Representational State Transfer)
2. GraphQL

## Analysis

### REST

REST (Representational State Transfer)[^3] uses HTTP methods and URLs to represent resources. Each endpoint returns a fixed data structure.

**Strengths:**

- **HTTP caching works out of the box.** GET requests cache naturally at CDN, browser, and proxy layers[^4]. Cache keys are just URLs.
- **Predictable performance.** Each endpoint has known, testable performance characteristics. You can optimise hot paths individually.
- **Mature tooling.** Monitoring, rate limiting, load balancing all work without special configuration.
- **Simple to understand.** Junior developers can be productive quickly. The mental model is straightforward.
- **Stateless by design.** Each request contains everything needed to process it[^5].

**Weaknesses:**

- **Over-fetching.** Endpoints return fixed shapes. Clients may receive data they don't need.
- **Under-fetching.** Complex views may require multiple round trips to assemble data.
- **Versioning overhead.** Breaking changes require new versions or careful deprecation.
- **Endpoint proliferation.** As requirements grow, you may end up with many specialised endpoints.

### GraphQL

GraphQL[^6] provides a query language that lets clients request exactly the data they need. Developed by Facebook and open-sourced in 2015[^7].

**Strengths:**

- **Client-specified queries.** Clients request exactly what they need, reducing over-fetching.
- **Single endpoint.** One endpoint serves all data needs. Simpler client configuration.
- **Strong typing.** The schema is self-documenting and enables excellent tooling[^8].
- **Reduced round trips.** Complex data requirements can be fulfilled in a single request.
- **Evolvable schema.** Adding fields doesn't break existing clients.

**Weaknesses:**

- **N+1 query problem.**[^9] The most common performance killer. A query for users with their orders can generate hundreds of database queries.

```graphql
# This innocent query...
query {
  users {
    name
    orders {
      total
    }
  }
}

# ...can generate:
# 1 query for users
# N queries for orders (one per user)
```

DataLoader[^10] or similar batching is required, but adds complexity and doesn't solve all cases.

- **Query complexity is unbounded.** Clients can construct expensive queries that overwhelm your server.

```graphql
# A malicious or naive client can do this
query {
  users {
    friends {
      friends {
        friends {
          orders {
            items {
              product {
                reviews {
                  author {
                    orders { ... }
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

You need query cost analysis[^11], depth limiting, and complexity scoring. This is non-trivial to implement correctly.

- **Caching is hard.** POST requests don't cache at HTTP layer. You need application-level caching (Apollo Client[^12], Relay[^13]) or persisted queries[^14].
- **Monitoring is harder.** All requests hit one endpoint. You need GraphQL-aware tooling to understand which queries are slow[^15].
- **Error handling is inconsistent.** Partial success (some fields resolve, some error) is valid in GraphQL. Clients must handle this complexity.
- **Performance problems are hidden.** REST exposes performance per endpoint. GraphQL hides it behind a flexible query interface. Problems surface later and are harder to diagnose[^16].
- **N+1 at the resolver level.** Even with DataLoader, resolvers can trigger additional N+1 patterns that are hard to spot.

## Performance Deep Dive

### Why GraphQL Performance Issues Are Common

1. **Complexity moved, not eliminated.** REST's "over-fetching" means the server does predictable work. GraphQL moves the complexity to the server, where every query is potentially unique[^17].

2. **Resolver architecture.** GraphQL resolvers[^18] are called per-field. Without careful design, this leads to repeated database calls.

3. **No query plan optimisation.** SQL databases optimise query plans. GraphQL servers execute resolvers independently with no cross-resolver optimisation.

4. **Nested pagination.** Paginating nested relationships is complex and often inefficient[^19].

```graphql
# How do you efficiently paginate orders within paginated users?
query {
  users(first: 10) {
    orders(first: 5) {
      items(first: 10) { ... }
    }
  }
}
```

### Mitigations (and Their Costs)

| Problem | Mitigation | Added Complexity |
|---------|------------|------------------|
| N+1 queries | DataLoader | Batching logic in every resolver |
| Expensive queries | Query cost analysis | Cost calculation rules, enforcement |
| Deep nesting | Depth limiting | Configuration, error handling |
| Caching | Persisted queries | Build pipeline, query allowlist |
| Monitoring | GraphQL APM tools | Additional tooling, cost |

Each mitigation is solvable, but the cumulative complexity is substantial.

## When to Choose What

### Choose REST when:

- You control both client and server
- Caching is critical (CDN, browser cache)
- Your data model maps naturally to resources
- Team is new to API development
- You need predictable, optimisable performance
- Public API with third-party consumers
- High-traffic endpoints with simple data shapes

### Choose GraphQL when:

- Multiple clients with different data needs (web, mobile, third-party)
- Rapid frontend iteration is prioritised over backend simplicity
- Your data is highly interconnected (social graphs, content management)
- You have the expertise to implement it correctly
- You're willing to invest in tooling and monitoring
- Mobile clients where bandwidth matters significantly

### Hybrid Approach

Some teams use both:

- REST for simple, high-traffic, cacheable endpoints
- GraphQL for complex, client-specific data requirements

This adds operational complexity but can be pragmatic.

## Common Anti-Patterns

### GraphQL over REST

Wrapping existing REST APIs with a GraphQL layer adds latency and complexity without solving the underlying data fetching problems.

### GraphQL as a database proxy

Exposing your database schema directly as a GraphQL schema leads to security and performance issues. Your GraphQL schema should reflect your domain model, not your database.

### Ignoring query complexity

Launching without query cost analysis is asking for production incidents. Malicious or naive clients will find your expensive queries.

### Assuming GraphQL is faster

GraphQL reduces over-fetching for clients but often increases server-side work. The network savings may not offset the backend costs.

## Decision Outcome

**For most API projects, REST remains the pragmatic default.** It's simpler, performs predictably, caches naturally, and has mature tooling.

**Consider GraphQL when you have specific requirements** that justify its complexity: diverse clients, highly connected data, and a team experienced enough to handle the pitfalls.

The choice is not about which technology is "better" but which trade-offs align with your constraints.

---

## References

[^1]: Fielding, Roy Thomas. (2000). "Architectural Styles and the Design of Network-based Software Architectures." Doctoral dissertation, University of California, Irvine. Chapter 5: REST. https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
[^2]: GraphQL Foundation. "GraphQL: A query language for your API." https://graphql.org/
[^3]: Fielding, Roy Thomas. (2000). "Architectural Styles and the Design of Network-based Software Architectures." Doctoral dissertation, University of California, Irvine. https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm
[^4]: Fielding, R. et al. (2014). "Hypertext Transfer Protocol (HTTP/1.1): Caching." RFC 7234, IETF. https://datatracker.ietf.org/doc/html/rfc7234
[^5]: Fielding, Roy Thomas. (2000). "Stateless." REST Architectural Constraints. https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_3
[^6]: GraphQL Foundation. "GraphQL Specification." https://spec.graphql.org/
[^7]: Lee, Byron. (2015). "GraphQL: A data query language." Facebook Engineering Blog. https://engineering.fb.com/2015/09/14/core-infra/graphql-a-data-query-language/
[^8]: GraphQL Foundation. "GraphQL Introspection." https://graphql.org/learn/introspection/
[^9]: Shopify Engineering. "Solving the N+1 Problem for GraphQL through Batching." https://shopify.engineering/solving-the-n-1-problem-for-graphql-through-batching
[^10]: GraphQL Foundation. "DataLoader - Batching and caching for GraphQL." https://github.com/graphql/dataloader
[^11]: Hype. "GraphQL Query Cost Analysis." https://github.com/slicknode/graphql-query-complexity
[^12]: Apollo GraphQL. "Apollo Client." https://www.apollographql.com/docs/react/
[^13]: Meta. "Relay - A JavaScript framework for building data-driven React applications." https://relay.dev/
[^14]: Apollo GraphQL. "Automatic Persisted Queries." https://www.apollographql.com/docs/apollo-server/performance/apq/
[^15]: Apollo GraphQL. "Why GraphQL Performance Monitoring is Hard." https://www.apollographql.com/blog/graphql/performance/why-graphql-performance-monitoring-is-hard/
[^16]: LogRocket. "GraphQL Performance Issues and How to Handle Them." https://blog.logrocket.com/graphql-performance-issues-and-how-to-handle-them/
[^17]: Biehl, Matthias. (2018). "GraphQL API Design." API University. https://api-university.com/books/graphql-api-design/
[^18]: GraphQL Foundation. "Execution - Resolvers." https://graphql.org/learn/execution/
[^19]: Relay. "GraphQL Cursor Connections Specification." https://relay.dev/graphql/connections.htm

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: Payloads and Errors](06-payloads-and-errors.md) | [Next: API Gateways](08-api-gateways.md)
