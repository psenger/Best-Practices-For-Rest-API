# API Communication Patterns

[Index](../README.md) | [Previous: Payloads and Errors](06-payloads-and-errors.md)

---

When designing APIs, teams often debate between REST and GraphQL. But the real question is broader: which communication pattern fits your problem? REST, GraphQL, WebSockets, and Server-Sent Events (SSE) each solve different problems and come with different trade-offs.

## Table of Contents

1. [Decision Drivers](#decision-drivers)
2. [REST](#rest)
3. [GraphQL](#graphql)
4. [WebSockets](#websockets)
5. [Server-Sent Events (SSE)](#server-sent-events-sse)
6. [Comparison Matrix](#comparison-matrix)
7. [When to Choose What](#when-to-choose-what)
8. [Hybrid Architectures](#hybrid-architectures)
9. [Common Anti-Patterns](#common-anti-patterns)
10. [Red Flags](#red-flags)

---

## Decision Drivers

Before choosing a pattern, consider:

- Is data flow **unidirectional** (server → client) or **bidirectional** (both ways)?
- Is the interaction **request-response** (client asks, server answers) or **event-driven** (server pushes when something happens)?
- How critical is **real-time latency**? (milliseconds vs seconds vs acceptable polling interval)
- What are the **caching requirements**?
- What is the **client landscape**? (browsers, mobile, IoT, server-to-server)
- What is the **infrastructure reality**? (load balancers, proxies, firewalls that may not support persistent connections)
- What is the **team's expertise**?

---

## REST

REST (Representational State Transfer)[^1] uses HTTP methods and resource URLs. Each endpoint returns a fixed data structure.

**Strengths:**

- **HTTP caching works out of the box.** GET requests cache naturally at CDN, browser, and proxy layers[^2]. Cache keys are just URLs.
- **Predictable performance.** Each endpoint has known, testable performance characteristics. You can optimise hot paths individually.
- **Mature tooling.** Monitoring, rate limiting, load balancing all work without special configuration.
- **Simple to understand.** Junior developers can be productive quickly. The mental model is straightforward.
- **Stateless by design.** Each request contains everything needed to process it[^3].

**Weaknesses:**

- **Over-fetching.** Endpoints return fixed shapes. Clients may receive data they don't need.
- **Under-fetching.** Complex views may require multiple round trips to assemble data.
- **Versioning overhead.** Breaking changes require new versions or careful deprecation.
- **Endpoint proliferation.** As requirements grow, you may end up with many specialised endpoints.
- **No server push.** Clients must poll for updates, which is inefficient for real-time data.

---

## GraphQL

GraphQL[^4] provides a query language that lets clients request exactly the data they need. Developed by Facebook and open-sourced in 2015[^5].

**Strengths:**

- **Client-specified queries.** Clients request exactly what they need, reducing over-fetching.
- **Single endpoint.** One endpoint serves all data needs. Simpler client configuration.
- **Strong typing.** The schema is self-documenting and enables excellent tooling[^6].
- **Reduced round trips.** Complex data requirements can be fulfilled in a single request.
- **Evolvable schema.** Adding fields doesn't break existing clients.
- **Subscriptions.** GraphQL has a built-in subscription model for real-time data over WebSockets.

**Weaknesses:**

- **N+1 query problem.**[^7] The most common performance killer. A query for users with their orders can generate hundreds of database queries.

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

DataLoader[^8] or similar batching is required, but adds complexity and doesn't solve all cases.

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

You need query cost analysis[^9], depth limiting, and complexity scoring.

- **Caching is hard.** POST requests don't cache at HTTP layer. You need application-level caching (Apollo Client[^10], Relay[^11]) or persisted queries[^12].
- **Monitoring is harder.** All requests hit one endpoint. You need GraphQL-aware tooling to understand which queries are slow[^13].
- **Error handling is inconsistent.** Partial success (some fields resolve, some error) is valid in GraphQL. Clients must handle this complexity.
- **Performance problems are hidden.** REST exposes performance per endpoint. GraphQL hides it behind a flexible query interface. Problems surface later and are harder to diagnose[^14].

### GraphQL Performance Deep Dive

1. **Complexity moved, not eliminated.** REST's "over-fetching" means the server does predictable work. GraphQL moves the complexity to the server, where every query is potentially unique[^15].

2. **Resolver architecture.** GraphQL resolvers[^16] are called per-field. Without careful design, this leads to repeated database calls.

3. **No query plan optimisation.** SQL databases optimise query plans. GraphQL servers execute resolvers independently with no cross-resolver optimisation.

4. **Nested pagination.** Paginating nested relationships is complex and often inefficient[^17].

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

**Mitigations and their costs:**

| Problem | Mitigation | Added Complexity |
|---------|------------|------------------|
| N+1 queries | DataLoader | Batching logic in every resolver |
| Expensive queries | Query cost analysis | Cost calculation rules, enforcement |
| Deep nesting | Depth limiting | Configuration, error handling |
| Caching | Persisted queries | Build pipeline, query allowlist |
| Monitoring | GraphQL APM tools | Additional tooling, cost |

---

## WebSockets

WebSockets[^18] (RFC 6455) provide a persistent, full-duplex communication channel over a single TCP connection. After an HTTP upgrade handshake, both client and server can send messages at any time.

**Strengths:**

- **Bidirectional.** Both client and server can push messages without request-response overhead.
- **Low latency.** No HTTP overhead per message after the initial handshake. Sub-millisecond delivery possible.
- **Real-time.** Ideal for data that changes continuously and must be reflected immediately.
- **Efficient for high-frequency updates.** Chat, gaming, collaborative editing, live trading, IoT telemetry.

**Weaknesses:**

- **Stateful connections.** Each client holds an open connection. This complicates load balancing, scaling, and deployment (rolling restarts drop connections).
- **No HTTP caching.** Messages bypass the HTTP caching layer entirely.
- **Firewall/proxy issues.** Some corporate firewalls, proxies, and older load balancers don't handle WebSocket upgrades correctly.
- **Scaling complexity.** Sticky sessions or a pub/sub backbone (Redis, NATS, Kafka) needed to fan out messages across server instances.
- **No built-in reconnection.** Clients must implement reconnection logic with backoff. The server has no way to "replay" missed messages without additional infrastructure.
- **Harder to monitor.** Persistent connections don't produce the same access log patterns as HTTP. Specialised tooling is required.
- **Security surface.** Persistent connections can be used for slow-read DoS attacks, and each open connection consumes server resources.

**Design checklist:**

- Is there a reconnection strategy with exponential backoff?
- How are missed messages handled during disconnection? (message queue, sequence numbers, event replay)
- Is there a heartbeat/ping-pong mechanism to detect stale connections?
- How does the design handle horizontal scaling? (sticky sessions, pub/sub backbone)
- Is authentication handled at connection time AND periodically revalidated?
- Are connection limits defined per user/IP to prevent resource exhaustion?

---

## Server-Sent Events (SSE)

SSE[^19] (EventSource API) provides a unidirectional, server-to-client push channel over standard HTTP. The server sends a stream of events; the client listens.

**Strengths:**

- **Simple.** Built on standard HTTP. Works through firewalls, proxies, and load balancers that support HTTP/1.1 or HTTP/2.
- **Auto-reconnection.** The browser's EventSource API automatically reconnects with `Last-Event-ID`, and the server can resume from where it left off. This is built into the spec, not bolted on.
- **HTTP/2 multiplexing.** Multiple SSE streams can share a single TCP connection, eliminating the browser's 6-connection-per-domain limit of HTTP/1.1.
- **Text-based protocol.** Easy to debug with standard HTTP tools (curl, browser dev tools).
- **Standard HTTP auth.** Cookies, headers, and existing auth infrastructure work as-is.
- **Efficient for server-push patterns.** Notifications, live feeds, progress updates, real-time dashboards.

**Weaknesses:**

- **Unidirectional only.** Server to client. The client cannot send data over the SSE connection — use REST/fetch for client-to-server.
- **Text only.** No binary data support (WebSockets support binary frames). Must base64-encode binary data, which adds overhead.
- **HTTP/1.1 connection limit.** Browsers allow only ~6 connections per domain on HTTP/1.1. Each SSE stream uses one. This is resolved by HTTP/2 but may matter for legacy deployments.
- **No native mobile support.** The EventSource API is a browser standard. Mobile apps need a library or custom implementation.
- **Smaller ecosystem.** Fewer libraries and frameworks compared to WebSockets.

**Design checklist:**

- Is the data flow truly unidirectional (server → client only)?
- Is `Last-Event-ID` used for resumption after reconnection?
- Is HTTP/2 available? (eliminates the connection limit issue)
- Are events structured with `id`, `event`, and `data` fields?
- Is there a keep-alive mechanism? (send comments `: keep-alive\n\n` to prevent proxy timeouts)

---

## Comparison Matrix

| Aspect | REST | GraphQL | WebSockets | SSE |
|--------|------|---------|------------|-----|
| **Direction** | Request-response | Request-response (+ subscriptions) | Bidirectional | Server → client |
| **Connection** | Stateless, new per request | Stateless (subscriptions are stateful) | Persistent, stateful | Persistent, stateful |
| **Caching** | Native HTTP caching | Application-level only | None | Standard HTTP |
| **Real-time** | Polling only | Via subscriptions | Native | Native |
| **Protocol** | HTTP | HTTP (subscriptions over WS) | WS (TCP after HTTP upgrade) | HTTP |
| **Binary data** | Yes (multipart) | No (base64) | Yes (binary frames) | No (text only) |
| **Firewall-friendly** | Yes | Yes | Sometimes problematic | Yes |
| **Browser support** | Universal | Via client libraries | Universal | Universal (EventSource) |
| **Scaling** | Stateless = easy | Stateless = easy (subscriptions hard) | Stateful = hard | Stateful but simpler than WS |
| **Reconnection** | N/A | N/A | Manual (client must implement) | Automatic (built into spec) |
| **Auth** | Per-request headers/cookies | Per-request headers/cookies | At handshake (revalidation needed) | Per-request headers/cookies |
| **Monitoring** | Standard HTTP tooling | Needs GraphQL-aware tools | Specialised tooling | Standard HTTP tooling |
| **Best for** | CRUD, public APIs, cacheable data | Complex queries, multi-client | Chat, gaming, collaboration | Notifications, feeds, dashboards |

---

## When to Choose What

### Choose REST when:

- Data is request-response (client asks, server answers)
- Caching is critical (CDN, browser cache)
- Your data model maps naturally to resources
- The team is new to API development
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

### Choose WebSockets when:

- Communication must be **bidirectional** (both client and server initiate messages)
- **Sub-second latency** is required (chat, gaming, live collaboration, trading)
- **High-frequency** updates in both directions (collaborative editing, multiplayer)
- The client needs to send data unprompted — not just in response to user action

### Choose SSE when:

- Data flow is **server-to-client only** (notifications, live feeds, progress updates)
- You want **automatic reconnection** with event replay — built into the spec
- You need **standard HTTP compatibility** (firewalls, proxies, auth, caching)
- Update frequency is moderate (seconds to minutes — not sub-millisecond)
- Simplicity is a goal: SSE is dramatically simpler to implement and operate than WebSockets

### Choose polling (REST) when:

- Updates are infrequent and latency tolerance is high (check every 30s–5min)
- Infrastructure doesn't support persistent connections
- Simplicity is paramount and the team has no real-time experience
- The cost of occasional stale data is acceptable

---

## Hybrid Architectures

Most production systems combine patterns. Common combinations:

**REST + SSE** — REST for CRUD operations, SSE for real-time notifications. The most common hybrid for web applications.

```
POST /orders          → REST (create order)
GET  /orders/{id}     → REST (read order)
GET  /orders/stream   → SSE  (live order status updates)
```

**REST + WebSockets** — REST for data operations, WebSockets for bidirectional real-time features (chat, collaboration).

```
GET  /messages        → REST (message history)
POST /messages        → REST (send message)
ws://api/chat         → WS   (real-time delivery + typing indicators)
```

**REST + GraphQL** — REST for simple, high-traffic, cacheable endpoints. GraphQL for complex, client-specific queries.

**GraphQL + Subscriptions** — GraphQL for queries/mutations, subscriptions (over WebSockets) for real-time updates. Apollo Server and Relay support this natively.

When mixing patterns, verify:

- Auth is consistent across patterns (same identity provider and tokens)
- Clear boundaries exist — which operations go through which pattern, documented
- The team can operate and monitor all the communication patterns in use

---

## Common Anti-Patterns

### GraphQL over REST

Wrapping existing REST APIs with a GraphQL layer adds latency and complexity without solving the underlying data fetching problems.

### GraphQL as a database proxy

Exposing your database schema directly as a GraphQL schema leads to security and performance issues. Your GraphQL schema should reflect your domain model, not your database.

### Ignoring query complexity

Launching without query cost analysis is asking for production incidents. Malicious or naive clients will find your expensive queries.

### Assuming GraphQL is faster

GraphQL reduces over-fetching for clients but often increases server-side work. The network savings may not offset the backend costs.

### WebSockets for unidirectional data

Using WebSockets when data only flows server-to-client. SSE is simpler, auto-reconnects, works through all proxies, and uses standard HTTP auth. WebSockets add complexity for no benefit in this case.

### WebSockets for infrequent updates

Using persistent connections for data that changes every few minutes. Polling with REST is simpler, stateless, and cacheable. Persistent connections have a cost — don't pay it for data that updates infrequently.

### Polling for real-time data

Using REST polling every second for data that needs sub-second delivery. This wastes bandwidth, hits rate limits, and still delivers stale data. Use SSE or WebSockets.

### No reconnection strategy

Deploying WebSockets without client-side reconnection logic, backoff, and missed-message handling. Connections will drop — the client must handle this gracefully.

---

## Red Flags

### In a REST design suggesting another pattern might be better:

- Many endpoints returning the same data in different shapes for different clients → **GraphQL**
- Clients consistently making 5+ requests to assemble a single view → **GraphQL**
- Frequent debates about "should we add this field to this endpoint?" → **GraphQL**
- Client polling an endpoint every 1–5 seconds for updates → **SSE**
- Requirement for "live" or "real-time" updates in a REST-only design → **SSE or WebSockets**

### In a GraphQL design suggesting REST might be better:

- All queries are simple resource lookups (no complex nesting)
- Caching is a primary concern
- The team has no GraphQL experience
- There's only one client consuming the API
- No plan for query cost analysis or depth limiting

### In a WebSocket design suggesting SSE might be better:

- Data flows server-to-client only (no client-initiated messages over the connection)
- The team is struggling with connection management, load balancing, or scaling
- Update frequency is seconds-to-minutes, not sub-second

### WebSockets are clearly the right choice when:

- Collaborative editing (multiple users editing simultaneously)
- Chat or messaging (bidirectional, low-latency)
- Live gaming (bidirectional, sub-millisecond)
- IoT device control (bidirectional command/telemetry)

---

## Decision Outcome

**For most API projects, REST remains the pragmatic default.** It's simpler, performs predictably, caches naturally, and has mature tooling.

**Add SSE** when you need server-push without bidirectional complexity.

**Add WebSockets** when you need true bidirectional real-time communication.

**Consider GraphQL** when you have specific requirements that justify its complexity: diverse clients, highly connected data, and a team experienced enough to handle the pitfalls.

The choice is not about which technology is "better" but which trade-offs align with your constraints. Most production systems use a combination.

---

## References

[^1]: Fielding, Roy Thomas. (2000). "Architectural Styles and the Design of Network-based Software Architectures." Doctoral dissertation, University of California, Irvine. Chapter 5: REST. https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
[^2]: Fielding, R. et al. (2014). "Hypertext Transfer Protocol (HTTP/1.1): Caching." RFC 7234, IETF. https://datatracker.ietf.org/doc/html/rfc7234
[^3]: Fielding, Roy Thomas. (2000). "Stateless." REST Architectural Constraints. https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_3
[^4]: GraphQL Foundation. "GraphQL: A query language for your API." https://graphql.org/
[^5]: Lee, Byron. (2015). "GraphQL: A data query language." Facebook Engineering Blog. https://engineering.fb.com/2015/09/14/core-infra/graphql-a-data-query-language/
[^6]: GraphQL Foundation. "GraphQL Introspection." https://graphql.org/learn/introspection/
[^7]: Shopify Engineering. "Solving the N+1 Problem for GraphQL through Batching." https://shopify.engineering/solving-the-n-1-problem-for-graphql-through-batching
[^8]: GraphQL Foundation. "DataLoader - Batching and caching for GraphQL." https://github.com/graphql/dataloader
[^9]: Hype. "GraphQL Query Cost Analysis." https://github.com/slicknode/graphql-query-complexity
[^10]: Apollo GraphQL. "Apollo Client." https://www.apollographql.com/docs/react/
[^11]: Meta. "Relay - A JavaScript framework for building data-driven React applications." https://relay.dev/
[^12]: Apollo GraphQL. "Automatic Persisted Queries." https://www.apollographql.com/docs/apollo-server/performance/apq/
[^13]: Apollo GraphQL. "Why GraphQL Performance Monitoring is Hard." https://www.apollographql.com/blog/graphql/performance/why-graphql-performance-monitoring-is-hard/
[^14]: LogRocket. "GraphQL Performance Issues and How to Handle Them." https://blog.logrocket.com/graphql-performance-issues-and-how-to-handle-them/
[^15]: Biehl, Matthias. (2018). "GraphQL API Design." API University. https://api-university.com/books/graphql-api-design/
[^16]: GraphQL Foundation. "Execution - Resolvers." https://graphql.org/learn/execution/
[^17]: Relay. "GraphQL Cursor Connections Specification." https://relay.dev/graphql/connections.htm
[^18]: Fette, I. and Melnikov, A. (2011). "The WebSocket Protocol." RFC 6455, IETF. https://datatracker.ietf.org/doc/html/rfc6455
[^19]: WHATWG. "Server-sent events." https://html.spec.whatwg.org/multipage/server-sent-events.html

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: Payloads and Errors](06-payloads-and-errors.md) | [Next: API Gateways](08-api-gateways.md)
