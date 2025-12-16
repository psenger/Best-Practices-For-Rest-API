# Resilience

[Index](../README.md) | [Previous: Design Principles](04-design-principles.md)

---

APIs fail. Networks drop. Services crash. Resilience is about designing systems that handle failure gracefully rather than catastrophically.

## Service Unavailable

Dynamic horizontal scaling services may experience brief unavailability as they come online. Both clients and servers need strategies to handle this gracefully.

Return `503 Service Unavailable` (not 504, which indicates a gateway timeout) with a `Retry-After` header[^1] when the service is temporarily unavailable.

## Exponential Backoff

Clients should implement retry logic with exponential backoff and jitter[^2]:

```javascript
async function fetchWithRetry(url, options, maxRetries = 5) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      const response = await fetch(url, options);
      if (response.status !== 503 && response.status !== 429) {
        return response;
      }

      const retryAfter = response.headers.get('Retry-After');
      if (retryAfter) {
        await sleep(parseInt(retryAfter) * 1000);
      } else {
        await sleepWithJitter(attempt);
      }
    } catch (err) {
      if (attempt === maxRetries - 1) throw err;
      await sleepWithJitter(attempt);
    }
  }
  throw new Error('Max retries exceeded');
}

function sleepWithJitter(attempt) {
  const baseDelay = Math.pow(2, attempt) * 1000; // 1s, 2s, 4s, 8s, 16s
  const jitter = Math.random() * 1000;           // 0-1s random jitter
  return new Promise(r => setTimeout(r, baseDelay + jitter));
}
```

**Why jitter matters:** Without jitter, when a service recovers from an outage, all waiting clients retry simultaneously, causing a thundering herd[^3] that brings the service down again.

## Circuit Breaker Pattern

The circuit breaker[^4] prevents cascading failures by failing fast when a downstream service is unhealthy.

**States:**

- **Closed** - Normal operation. Requests pass through. Track failure rate.
- **Open** - Service is down. Fail immediately without calling downstream. Return cached data or graceful degradation.
- **Half-Open** - After a timeout, allow a few test requests through. If they succeed, close the circuit. If they fail, reopen.

```
         success
            │
    ┌───────┴───────┐
    │               │
    ▼               │
┌────────┐    ┌─────┴────┐    timeout    ┌───────────┐
│ Closed │───▶│   Open   │────────-─────▶│ Half-Open │
└────────┘    └──────────┘               └───────────┘
  failure        │                            │
  threshold      │         failure            │
  reached        ◀────────────────────────────┘
                              │
                              │ success
                              ▼
                         back to Closed
```

**Configuration:**

- **Failure threshold** - How many failures before opening (e.g., 5 failures in 60 seconds)
- **Open duration** - How long to stay open before trying half-open (e.g., 30 seconds)
- **Half-open limit** - How many test requests to allow (e.g., 3 requests)

```javascript
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5;
    this.resetTimeout = options.resetTimeout || 30000;
    this.state = 'CLOSED';
    this.failures = 0;
    this.lastFailure = null;
  }

  async call(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailure > this.resetTimeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is open');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (err) {
      this.onFailure();
      throw err;
    }
  }

  onSuccess() {
    this.failures = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failures++;
    this.lastFailure = Date.now();
    if (this.failures >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
}
```

Libraries like opossum[^5] (Node.js), resilience4j[^6] (Java), or Polly[^7] (.NET) provide production-ready implementations.

## Graceful Degradation

When circuits open, don't just fail. Degrade gracefully:

- Return cached data (even if stale)
- Return partial results
- Use fallback services
- Queue requests for later processing

The goal is to keep the user experience functional, even if imperfect. Netflix's approach to graceful degradation[^8] is worth studying.

## Timeouts

Every external call needs a timeout. Without timeouts, a slow downstream service can exhaust your connection pool and bring down your entire system.

**Timeout guidelines:**

- **Connect timeout** - How long to wait for a connection (1-5 seconds)
- **Read timeout** - How long to wait for a response (varies by operation)
- **Total timeout** - Maximum time for the entire operation including retries

```javascript
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch(url, { signal: controller.signal });
} finally {
  clearTimeout(timeout);
}
```

Set timeouts based on your SLAs. If you promise p99 latency of 200ms, your downstream timeouts need to be well under that.

## Bulkheads

Bulkheads[^9] isolate failures to prevent them from spreading. Named after ship compartments that contain flooding.

**Implementation approaches:**

- **Thread pool isolation** - Each dependency gets its own thread pool. If one fills up, others continue working.
- **Connection pool isolation** - Separate connection pools per service.
- **Semaphore isolation** - Limit concurrent requests to each dependency.

```javascript
class Bulkhead {
  constructor(maxConcurrent) {
    this.maxConcurrent = maxConcurrent;
    this.current = 0;
    this.queue = [];
  }

  async execute(fn) {
    if (this.current >= this.maxConcurrent) {
      throw new Error('Bulkhead full');
    }

    this.current++;
    try {
      return await fn();
    } finally {
      this.current--;
    }
  }
}
```

If your payment service is slow, it shouldn't take down your product catalog.

## Caching

The inevitability of caching. I have worked on systems with so much caching wired between all the layers that it became impossible to find who was holding on to what. Caches at the CDN, the gateway, the service, the ORM, the database. Each layer trying to be "helpful." The result is stale data, mysterious inconsistencies, and debugging nightmares.

**The pragmatic approach:** Provide sensible caching at service boundaries that can be controlled by standard HTTP headers[^10]. This satisfies consumers who want performance while keeping the system of record authoritative.

### Cache-Control Headers

Use standard HTTP caching headers rather than inventing your own:

```http
Cache-Control: max-age=3600, must-revalidate
ETag: "abc123"
Last-Modified: Wed, 21 Oct 2024 07:28:00 GMT
Vary: Accept-Encoding, Authorization
```

**Key directives:**

| Directive | Purpose |
|-----------|---------|
| `max-age` | How long (seconds) the response can be cached |
| `no-cache` | Must revalidate with origin before using cached copy |
| `no-store` | Never cache this response (sensitive data) |
| `private` | Only browser can cache, not shared caches (CDNs) |
| `public` | Any cache can store this response |
| `must-revalidate` | Once stale, must check origin before using |

### What to Cache

**Good candidates for caching:**

- Reference data that changes infrequently (country lists, currency codes)
- Public content (product catalogs, published articles)
- Computed results that are expensive to generate
- Static assets (images, documents)

**Never cache:**

- User-specific data without proper `Vary` headers
- Sensitive information (use `Cache-Control: no-store`)
- Data from the system of record that must be current (account balances, inventory counts)
- Responses to POST, PUT, PATCH, DELETE (by HTTP specification)

### Caching Layers

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│ Browser │───▶│   CDN   │───▶│ Gateway │───▶│ Service │───▶│   DB    │
└─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘
   Cache 1       Cache 2       Cache 3        Cache 4       The Truth
```

**The rule:** The closer to the user, the less authoritative. The closer to the database, the more current. Design your cache strategy with this in mind.

### Cache Invalidation

"There are only two hard things in Computer Science: cache invalidation and naming things." - Phil Karlton[^11]

**Strategies:**

1. **Time-based expiry (TTL)** - Simple but may serve stale data
2. **Event-driven invalidation** - Publish events when data changes, caches subscribe
3. **Conditional requests** - Use ETags[^12] to check if cached data is still valid

```http
GET /products/123
If-None-Match: "abc123"

# If still valid:
HTTP/1.1 304 Not Modified

# If changed:
HTTP/1.1 200 OK
ETag: "def456"
{...new data...}
```

### Avoid Cache Stampedes

When a popular cached item expires, many requests hit the origin simultaneously. Strategies to prevent this:

- **Stale-while-revalidate**[^13] - Serve stale data while refreshing in background
- **Lock-based refresh** - Only one request refreshes, others wait
- **Probabilistic early refresh** - Randomly refresh before expiry

```http
Cache-Control: max-age=3600, stale-while-revalidate=60
```

This allows the cache to serve stale data for up to 60 seconds while fetching a fresh copy.

### Redis as a Cache

For service-level caching, Redis[^14] is the common choice:

```javascript
async function getCachedProduct(id) {
  const cached = await redis.get(`product:${id}`);
  if (cached) {
    return JSON.parse(cached);
  }

  const product = await db.products.findById(id);
  await redis.setex(`product:${id}`, 3600, JSON.stringify(product));
  return product;
}
```

**Guidelines:**

- Set TTLs on everything (don't let caches grow unbounded)
- Use consistent key naming (`{entity}:{id}`)
- Consider using hash structures for related data
- Monitor cache hit rates; low rates suggest wrong strategy

### Debugging Caching Issues

When you can't find "who's holding onto what":

1. **Add cache headers to responses** - Include `X-Cache: HIT` or `X-Cache: MISS`
2. **Log cache interactions** - Know when data is served from cache vs origin
3. **Use unique request IDs** - Trace a request through all layers
4. **Document your cache topology** - Everyone should know where caches exist

The goal is transparency: anyone debugging the system should be able to determine where data came from and how fresh it is.

---

## References

[^1]: Nottingham, M. and Fielding, R. (2012). "Additional HTTP Status Codes." RFC 6585, IETF. https://datatracker.ietf.org/doc/html/rfc6585
[^2]: AWS. "Exponential Backoff and Jitter." AWS Architecture Blog. https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
[^3]: Wikipedia. "Thundering herd problem." https://en.wikipedia.org/wiki/Thundering_herd_problem
[^4]: Fowler, Martin. (2014). "CircuitBreaker." https://martinfowler.com/bliki/CircuitBreaker.html
[^5]: Nodeshift. "Opossum - Circuit Breaker for Node.js." https://github.com/nodeshift/opossum
[^6]: Resilience4j. "Fault tolerance library for Java." https://github.com/resilience4j/resilience4j
[^7]: App-vNext. "Polly - .NET resilience and transient-fault-handling library." https://github.com/App-vNext/Polly
[^8]: Netflix. "Making the Netflix API More Resilient." Netflix Tech Blog. https://netflixtechblog.com/making-the-netflix-api-more-resilient-a8ec62f4b4f7
[^9]: Microsoft. "Bulkhead pattern." Azure Architecture Patterns. https://learn.microsoft.com/en-us/azure/architecture/patterns/bulkhead
[^10]: Fielding, R. et al. (2014). "Hypertext Transfer Protocol (HTTP/1.1): Caching." RFC 7234, IETF. https://datatracker.ietf.org/doc/html/rfc7234
[^11]: Attributed to Phil Karlton. https://martinfowler.com/bliki/TwoHardThings.html
[^12]: Fielding, R. and Reschke, J. (2014). "Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests." RFC 7232, IETF. https://datatracker.ietf.org/doc/html/rfc7232
[^13]: Nottingham, M. (2010). "HTTP Cache-Control Extensions for Stale Content." RFC 5861, IETF. https://datatracker.ietf.org/doc/html/rfc5861
[^14]: Redis. "In-memory data structure store." https://redis.io/

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: Design Principles](04-design-principles.md) | [Next: Payloads and Errors](06-payloads-and-errors.md)
