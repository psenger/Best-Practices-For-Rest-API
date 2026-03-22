# API Extensibility and Evolution

[Index](../README.md) | [Previous: Glossary](09-glossary.md)

---

How many parameters an endpoint accepts (its "arity") and whether the payload shape is fixed or open has a profound impact on API sustainability, backwards compatibility, and long-term maintainability. This chapter explores how to design APIs that grow gracefully.

## Table of Contents

1. [Fixed vs Variable Arity](#fixed-vs-variable-arity)
2. [Metadata as an Extension Point](#metadata-as-an-extension-point)
3. [Response Evolution](#response-evolution)
4. [SOLID Principles Applied to APIs](#solid-principles-applied-to-apis)
5. [Postel's Law](#postels-law)
6. [Hyrum's Law](#hyrums-law)
7. [The Design Pattern](#the-design-pattern)
8. [Common Gaps to Flag](#common-gaps-to-flag)

---

## Fixed vs Variable Arity

**Fixed arity** — the endpoint accepts a known, documented set of parameters. The request shape is fully specified in the OpenAPI contract.

```
POST /v1/customers
{ "name": "Alice", "email": "alice@example.com", "description": "VIP" }
```

**Variable arity** — the endpoint accepts a dynamic or open-ended set of parameters (query parameter bags, filter objects, extensible bodies).

```
GET /v1/issues?labels=bug,enhancement&state=open&assignee=alice&sort=updated&per_page=30
```

| Aspect | Fixed Arity | Variable Arity |
|--------|-------------|----------------|
| **Validation** | Strong, schema-enforceable | Harder to validate |
| **Documentation** | Clear, exhaustive | Harder to cover all combinations |
| **Code generation** | Excellent (typed clients) | Requires conventions |
| **Evolution** | Requires versioning to add params | Can add optional params without breaking |
| **Type safety** | Strong | Weak for the variable parts |
| **Discoverability** | High | Lower — consumers must know conventions |

**Practical guidance:**

- Use fixed arity for core write operations (create, update, delete) where strong validation matters.
- Use variable arity for search and filter operations where consumer needs are diverse and evolving.
- When adding parameters, make them optional with sensible defaults. Never add a required parameter to an existing endpoint — that's a breaking change.

---

## Metadata as an Extension Point

A `metadata` field is an intentionally schema-less key-value map that lets consumers store arbitrary data without the API provider changing the contract. This is the primary pattern for API extensibility without versioning.

**Real-world examples:**

- **Stripe**[^1] — `metadata` on every object (Customer, PaymentIntent, Subscription). Up to 50 keys, key names up to 40 chars, values up to 500 chars. Explicitly documented as having no functional impact on Stripe's behaviour.
- **AWS**[^2] — Resource "tags" across every service. Up to 50 tags per resource. Used for cost allocation, access control policies, and automation — despite being schema-less.
- **Slack**[^3] — Structured `metadata` with `event_type` and `event_payload`. Optionally validated against a registered schema — a hybrid approach.

```json
{
  "id": "cus_abc123",
  "name": "Alice",
  "email": "alice@example.com",
  "metadata": {
    "cms_id": "6573",
    "plan_tier": "enterprise",
    "referred_by": "user_42"
  }
}
```

**Design checklist:**

- Do API objects include a `metadata` (or `tags`) extension point for consumer-owned data?
- Are metadata constraints defined? (max keys, max key length, max value length, string-only values)
- Is metadata documented as inert storage (no side effects) or as having functional impact?
- Is metadata searchable/filterable, or storage-only? (this shapes consumer expectations)
- In OpenAPI, is the field defined with `additionalProperties: { type: string }` and `maxProperties`?

---

## Response Evolution

The same principles apply to API responses. A response with a rigid, fixed shape is predictable but inflexible. A response with extension points evolves without breaking consumers.

**Backwards-compatible vs breaking response changes:**

| Change | Safe? | Why |
|--------|-------|-----|
| Add new optional field | Yes | Consumers should ignore unknown fields (Postel's Law) |
| Add new value to an enum field | Risky | Consumers with exhaustive switch/case will break (Hyrum's Law) |
| Remove a field | No | Consumers depending on it will break |
| Change a field's type | No | Breaks deserialization in typed clients |
| Change field from always-present to sometimes-null | Risky | Consumers not handling null will break |
| Add nested object where there was a scalar | No | Breaks all consumers |

**Design checklist:**

- Are response schemas designed to be additive? (new fields can be added without breaking consumers)
- Are consumers expected to ignore unknown fields? (documented in API guidelines)
- Is field selection supported for large responses? (`?fields=id,name,email`)
- Are envelope wrappers used consistently? (`{ "data": {...}, "meta": {...} }` — provides a stable outer structure that can grow)
- Are enum values documented as potentially extensible? ("New values may be added — do not use exhaustive matching")
- Is the response shape consistent across similar endpoints? (all list endpoints use the same pagination structure)
- Do response objects include a `metadata` field for consumer-owned extension data?

**Stripe's approach to response evolution:**[^4]

- Adding new properties to responses is backwards-compatible
- Changing the order of properties is backwards-compatible
- Removing fields or changing types requires a new API version
- Consumers are version-pinned and opt into new shapes explicitly

---

## SOLID Principles Applied to APIs

SOLID[^5] applies to API design as much as to object-oriented code.

### Open/Closed Principle (OCP)

The API is "closed" for existing consumers but "open" for extension through optional parameters and metadata. Adding optional fields or parameters is backwards-compatible; removing or changing types requires a new version.

An API endpoint should be stable for existing callers while allowing the provider to extend it for new use cases — without forcing existing consumers to change.

### Interface Segregation Principle (ISP)

Each endpoint should accept only the parameters relevant to its concern. Avoid one fat endpoint that handles everything. A reporting endpoint that also accepts write parameters is a violation of ISP.

Consumers should not be forced to depend on parameters they don't use. Split by concern, not by convenience.

### Liskov Substitution Principle (LSP)

A new API version must be substitutable for the old. Preconditions (what the API requires of callers) cannot be strengthened; postconditions (what the API promises to return) cannot be weakened.

Concretely: if v1 accepts `name` as optional and returns `id` + `name`, then v2 cannot make `name` required (strengthened precondition) or stop returning `id` (weakened postcondition) without being a breaking change.

### Single Responsibility Principle (SRP)

Each endpoint should do one thing. An endpoint that creates a user, sends a welcome email, provisions a subscription, and charges a payment card is doing too much. If any step fails, what's the caller's recovery path?

### Dependency Inversion Principle (DIP)

API contracts should depend on abstractions (domain models), not on implementation details (database schemas, internal service names). Exposing `tbl_users.usr_email_addr` as a field name violates DIP.

---

## Postel's Law

"Be conservative in what you send, be liberal in what you accept."[^6]

Applied to APIs:

- **Liberal in what you accept:** Accept optional fields gracefully. Ignore unexpected fields in request bodies rather than rejecting them. Accept multiple date formats, case-insensitive enum values, trailing whitespace in strings.
- **Conservative in what you send:** Return a clean, documented, consistent response. Don't send undocumented fields, internal state, or debug information.

**The tension:** Being too liberal creates problems. If you accept and silently ignore a misspelled field (`recepients` instead of `recipients`), the caller gets no error but also gets no action. Bugs become invisible. Postel's Law is a principle of robustness, not a licence for sloppy validation.

The pragmatic balance: be liberal at the edges (input sanitisation, field trimming, format flexibility) and strict at the core (validate required fields, reject malformed data, return clear errors when validation fails).

---

## Hyrum's Law

"With a sufficient number of users of an API, it does not matter what you promise in the contract — all observable behaviours of your system will be depended on by somebody."[^7]

This is the biggest risk of metadata fields and variable arity. If consumers consistently store `metadata["order_id"]`, downstream systems will depend on it — it becomes a de facto contract even though it was never in the schema.

**Practical consequences:**

- Changing the order of fields in a JSON response will break consumers who parse positionally rather than by key.
- A response that always returns an empty array for "no results" instead of null becomes a contract — even if you documented null as valid.
- Adding a new enum value breaks consumers with exhaustive match statements, even though the OpenAPI spec said the field was extensible.

**Design implications:**

- Document observable behaviours explicitly. If consumers will depend on something, own it.
- Fixed arity reduces Hyrum's Law exposure; variable arity and metadata increase the surface area for implicit dependencies.
- When something works "by accident" at scale, that accident becomes the contract. Test explicitly for the behaviours you're promising, so unintended behaviours are caught before they become dependencies.
- When you need to change an accidental behaviour, provide a migration path: deprecation headers, versioned endpoints, changelog notices.

---

## The Design Pattern

The best APIs combine both: a **strict core schema** (fixed arity, typed, validated) with a **metadata escape hatch** (variable arity, untyped, consumer-owned). The core carries the contract; the metadata carries the extensibility.

```
RIGID ←——————————————————————————————→ FLEXIBLE
Fixed arity (core params)              Variable arity (metadata, filters)
Strong contracts, typed                Schema-less, unvalidated
Easy to version                        Hard to version (what's in metadata?)
Hyrum-resistant                        Hyrum-vulnerable
```

When a metadata pattern becomes universal across consumers, promote it to a first-class field in the next API version.

**Example lifecycle:**

1. Launch with `metadata` as a free-form field.
2. Observe that 80% of consumers store `metadata["account_tier"]`.
3. In v2, promote `account_tier` to a typed, validated first-class field.
4. Keep `metadata` for the next wave of consumer-specific extensions.

---

## Common Gaps to Flag

- No `metadata` or extension point on API objects — rigid schema, no room to grow without versioning
- No guidance for consumers on handling unknown response fields
- Enum fields documented as exhaustive with no plan for adding values
- Response shape changes treated casually — no additive-only policy
- Metadata used as a substitute for proper schema design ("just put it in metadata")
- No size constraints on metadata — unbounded storage and performance risk
- Metadata with hidden functional side effects — violates Principle of Least Astonishment
- Consumers storing sensitive data (PII, credentials) in metadata without encryption or access controls
- No plan for promoting common metadata patterns to first-class fields

---

## References

[^1]: Stripe. "Metadata." https://docs.stripe.com/metadata
[^2]: AWS. "Tag your Amazon EC2 resources." https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html
[^3]: Slack. "Message Metadata." https://api.slack.com/metadata/using
[^4]: Stripe. "API versioning." https://stripe.com/docs/api/versioning
[^5]: Martin, Robert C. (2000). "Design Principles and Design Patterns." https://web.archive.org/web/20150906155800/http://www.objectmentor.com/resources/articles/Principles_and_Patterns.pdf
[^6]: Postel, J. (1980). "Robustness Principle." RFC 761, IETF. https://datatracker.ietf.org/doc/html/rfc761
[^7]: Wright, Hyrum. "Hyrum's Law." https://www.hyrumslaw.com/

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: Glossary](09-glossary.md)
