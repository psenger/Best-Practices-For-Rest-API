# Payloads and Errors

[Index](../README.md) | [Previous: Resilience](05-resilience.md)

---

How you structure requests and responses matters. Consistent payloads reduce integration friction. Good error messages reduce support burden. See JSON API specification[^1] for one approach to standardising response structures.

## Response Structure

API responses should be deterministic: given the same input, return the same output.

### Envelope vs No Envelope

**With envelope:**
```json
{
  "data": [],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 150
  }
}
```

**Without envelope:**
```json
[
  { "id": "user_123", "name": "Alice" },
  { "id": "user_456", "name": "Bob" }
]
```
With headers: `X-Total-Count: 150`, `Link: </users?page=2>; rel="next"`

**Opinions vary.** Some prefer envelopes for consistency; others argue they add unnecessary nesting.

If you use envelopes:
- Keep `data` as an array for collections, object for single resources
- Include `meta` for pagination and other metadata
- Use Problem Details[^2] (RFC 9457) for errors rather than a custom `errors` field

Choose one approach and apply it consistently.

## Field Selection (Projection)

Allow clients to request only the fields they need[^3]. This reduces payload size and improves performance.

```
GET /users/123?fields=firstName,lastName,email
```

This is similar to GraphQL's field selection but for REST. Implement as:
- **Query parameter** - `?fields=firstName,lastName,email`
- **Custom header** - `X-Fields: firstName,lastName,email`

Use an additive approach (specify what to include) rather than exclusionary (specify what to exclude).

## Pagination

Pagination is essential for any API returning collections[^4]. The approach you choose affects performance, reliability, and client complexity.

### Offset-Based Pagination

Simple but has performance issues with large datasets[^5]:

```json
{
  "data": [],
  "page": 1,
  "pageSize": 50,
  "total": 2340
}
```

**Drawbacks:**
- `OFFSET 10000` is slow on most databases
- Insertions/deletions can cause items to be skipped or duplicated

### Cursor-Based Pagination

More efficient for large datasets and real-time data[^6]:

```json
{
  "data": [],
  "cursors": {
    "before": "abc123",
    "after": "xyz789"
  },
  "hasMore": true
}
```

Cursor-based pagination uses an opaque cursor (often a Base64-encoded identifier) rather than page numbers. This approach:
- Handles insertions/deletions gracefully
- Performs consistently regardless of offset depth
- Works well with real-time data streams

### Link-Based Pagination (HATEOAS)

Following HATEOAS principles[^7], include navigation links in responses:

```json
{
  "data": [],
  "links": {
    "self": "/users?page=2",
    "first": "/users?page=1",
    "prev": "/users?page=1",
    "next": "/users?page=3",
    "last": "/users?page=47"
  }
}
```

### Header-Based Pagination

Keeps the response body clean using the `Link` header per RFC 8288[^8]:

```
X-Page: 2
X-Page-Size: 50
X-Total-Count: 2340
Link: </users?page=1>; rel="prev", </users?page=3>; rel="next"
```

### When to Use What

| Approach | Best For |
|----------|----------|
| Offset pagination | Simple admin interfaces, small datasets |
| Cursor pagination | Infinite scroll, large datasets, real-time feeds |
| Link headers | Hypermedia-driven clients, keeping payload minimal |

## Errors - Problem Details (RFC 9457)

The Problem Details specification[^9] (originally RFC 7807[^10], superseded by RFC 9457 in 2023) provides a standardised approach to error responses. This format has been adopted by major APIs and frameworks.

### Standard Fields

| Field | Type | Description |
|-------|------|-------------|
| `type` | string (URI) | Identifies the problem type. When dereferenced, should provide human-readable documentation. Defaults to `about:blank`. This URI should never change, making it a stable identifier. |
| `title` | string | Short, human-readable summary. Should not change between occurrences except for localisation. |
| `status` | number | HTTP status code for this occurrence. |
| `detail` | string | Human-readable explanation specific to this occurrence. |
| `instance` | string (URI) | Identifies the specific occurrence of the problem. |

### Example: Business Logic Error

```http
HTTP/1.1 403 Forbidden
Content-Type: application/problem+json

{
  "type": "https://example.com/probs/out-of-credit",
  "title": "You do not have enough credit.",
  "status": 403,
  "detail": "Your current balance is 30, but that costs 50.",
  "instance": "/account/12345/msgs/abc",
  "balance": 30,
  "accounts": ["/account/12345", "/account/67890"]
}
```

### Example: Validation Error

```http
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json

{
  "type": "https://example.net/validation-error",
  "title": "Your request parameters didn't validate.",
  "status": 400,
  "detail": "Age must be a positive integer and color must be 'green', 'red', or 'blue'.",
  "errors": [
    {
      "field": "age",
      "code": "invalid_format",
      "message": "must be a positive integer"
    },
    {
      "field": "color",
      "code": "out_of_range",
      "message": "must be 'green', 'red', or 'blue'"
    }
  ]
}
```

You can extend Problem Details with custom fields (like `balance`, `accounts`, or `errors` above) as needed for your domain.

### Error Response Best Practices

For validation and domain errors, structure them to be actionable:

- **field** - The path to the problematic field (supports nested: `address.zipCode`)
- **code** - Machine-readable error code for programmatic handling
- **message** - Human-readable description

Don't expose internal details:
- No stack traces in production
- No database error messages
- No internal service names

## Identifiers

Use UUIDs[^11] or similar random identifiers rather than sequential integers:

| Sequential | UUID |
|------------|------|
| Predictable, easy to enumerate | Random, no information leakage |
| Reveals record count | No business intelligence exposed |
| Collisions in distributed systems | Globally unique |

### Type-Prefixed IDs

Type-prefixed IDs[^12] make debugging and logging easier. Companies like Stripe[^13] and Slack have popularised this pattern:

```
user_abc123def456
order_xyz789ghi012
txn_qrs456tuv789
```

Benefits:
- Immediately identify entity type from any ID
- Easier log searching and debugging
- Prevents accidentally using an order ID where a user ID is expected
- Works well with detached data (exported JSON, logs, error reports)

Always represent IDs as strings in JSON[^14], even if they're numeric internally. This ensures compatibility across systems and prevents JavaScript's number precision issues with large integers[^15]. Twitter famously encountered this when tweet IDs exceeded JavaScript's `Number.MAX_SAFE_INTEGER`[^16].

## Content Negotiation

Use `Accept` and `Content-Type` headers[^17] properly per HTTP semantics (RFC 9110[^18]):

```http
GET /users/123
Accept: application/json
```

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

Support multiple formats if needed, but JSON[^19] is the default. If you support XML, use the same structure as JSON (don't change field names or nesting).

---

## References

[^1]: JSON:API. "A specification for building APIs in JSON." https://jsonapi.org/
[^2]: Nottingham, M. et al. (2023). "Problem Details for HTTP APIs." RFC 9457, IETF. https://datatracker.ietf.org/doc/html/rfc9457
[^3]: Google Cloud. "API Design Guide - Standard Methods." https://cloud.google.com/apis/design/standard_methods
[^4]: Atlassian. "REST API Design Guidelines - Pagination." https://developer.atlassian.com/server/framework/atlassian-sdk/rest-api-design-guidelines-pagination/
[^5]: Percona. "Why OFFSET is slow." https://www.percona.com/blog/why-order-by-with-limit-and-offset-is-slow/
[^6]: Slack Engineering. (2020). "Evolving API Pagination at Slack." https://slack.engineering/evolving-api-pagination-at-slack/
[^7]: Fielding, Roy. (2008). "REST APIs must be hypertext-driven." https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven
[^8]: Nottingham, M. (2017). "Web Linking." RFC 8288, IETF. https://datatracker.ietf.org/doc/html/rfc8288
[^9]: Nottingham, M. et al. (2023). "Problem Details for HTTP APIs." RFC 9457, IETF. https://datatracker.ietf.org/doc/html/rfc9457
[^10]: Nottingham, M. and Wilde, E. (2016). "Problem Details for HTTP APIs." RFC 7807, IETF. https://datatracker.ietf.org/doc/html/rfc7807
[^11]: Leach, P. et al. (2005). "A Universally Unique IDentifier (UUID) URN Namespace." RFC 4122, IETF. https://datatracker.ietf.org/doc/html/rfc4122
[^12]: Stickfigure. "How to (and how not to) design REST APIs." https://github.com/stickfigure/blog/wiki/How-to-%28and-how-not-to%29-design-REST-APIs#rule-7-do-prefix-your-identifiers
[^13]: Stripe. "API Reference - Object IDs." https://stripe.com/docs/api
[^14]: Bray, T. (2017). "The JavaScript Object Notation (JSON) Data Interchange Format." RFC 8259, IETF. https://datatracker.ietf.org/doc/html/rfc8259
[^15]: MDN Web Docs. "Number.MAX_SAFE_INTEGER." https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER
[^16]: Twitter Developer Documentation. "Twitter IDs." https://developer.twitter.com/en/docs/twitter-ids
[^17]: MDN Web Docs. "Content negotiation." https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation
[^18]: Fielding, R. et al. (2022). "HTTP Semantics." RFC 9110, IETF. Section 12: Content Negotiation. https://httpwg.org/specs/rfc9110.html#content.negotiation
[^19]: Crockford, Douglas. "Introducing JSON." https://www.json.org/

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: Resilience](05-resilience.md) | [Next: GraphQL vs REST](07-graphql-vs-rest.md)
