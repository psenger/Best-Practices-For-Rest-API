# The Human Aspect

[Index](../README.md)

---

Service Oriented Architecture is a relatively new concept: "new" meaning within the last 25 years. Before REST[^1], we had Remote Procedure Calls popularised by CORBA[^2], the venerable EDI[^3], and later SOAP[^4]. The landscape has shifted repeatedly, and with each shift comes added complexity.

Given this churn, the biggest barrier to adoption isn't technical; it's human. Resistance shows up as laziness, being buried in other work, pure stubbornness, or the classic "we don't need to change, everything works fine."

1. Adoption
2. Standards and Consistency
3. Self Discovery
4. Intuitive

## Adoption

The number one challenge I have consistently had is adoption. While you might think it makes sense to add a layer of abstraction, apply the single responsibility principle[^5] to each layer, or create a stable elastic system to your total solution landscape, you will find that not everyone will agree.

This is usually founded in either the fact that it will inherently become more complex, or the users of your system enjoy the nature of traditional single layer systems. In either case, you will be forced to nurture the idea and advocate the adoption.

Although I can't guarantee adoption, there are several pillars needed for adoption: Ease of use, Documentation, Stability, and Assurance.

### Ease of Use

Adoption is always blocked when you can't sell the concept. Besides having an elevator pitch ready, you need to remove all barriers preventing adoption. These tend to be:

1. Lack of documentation
2. Lack of standards
3. Lack of convention
4. Difficulty in gaining access to resources

How can you address these things efficiently? Use the OpenAPI Specification[^6] (formerly Swagger) to define your API in a design-first approach. Tools like Swagger UI[^7], Redoc[^8], and Stoplight[^9] can generate interactive documentation from your OpenAPI definitions.

### Documentation

Documentation is one of the pillars of success. Lack thereof implies you haven't thought the system through or its ramifications. What kind of documentation should you create? API specifications, user stories, and examples.

Managing relevance of the documentation is difficult and requires discipline. Use OpenAPI 3.x[^10] to build a design-first approach. The added benefit is this specification can be used to create client and server stubs as well as create interactive websites. Tools like Swagger Codegen[^11] or OpenAPI Generator[^12] can generate code in dozens of languages. Because OpenAPI can create documentation and examples, this approach is invaluable.

### Stability

Stability means your API behaves predictably over time. Consumers should be able to rely on consistent behaviour without unexpected breaking changes. Define and publish your Non-Functional Requirements (NFRs):

- **Response time** - p50, p95, p99 latency targets (e.g., p99 < 200ms). See Google's SRE book on latency[^13].
- **Availability** - Uptime SLA (99.9%, 99.95%). Use an uptime calculator[^14] to understand the implications.
- **Throughput** - Requests per second the API can handle
- **Payload limits** - Maximum request/response sizes

When you must introduce breaking changes, communicate early and provide migration guides. Consider using Sunset headers per RFC 8594[^15] to signal deprecation.

### Assurance

Creating tests should be part of every developer's daily activity. It provides assurance that the goal was accomplished and meets the needs of the specification.

Validate your NFRs continuously:

- **Load testing** - Verify performance under expected and peak loads. Tools include k6[^16], Gatling[^17], and Locust[^18].
- **Synthetic monitoring** - Continuously probe endpoints to detect degradation. See Google's SRE book on monitoring[^19].
- **Chaos engineering** - Test failure modes before they happen in production. See the Principles of Chaos Engineering[^20].

## Intuitive

If you cannot explain your API in 30 seconds (the elevator pitch), it will be difficult to explain in writing let alone to others in documentation. Use web standards only where they make sense but **use the standards**. For example, a developer should be able to use a browser and point it at the service to see the results. JSON[^21] is the de facto standard for data format. This is a schema-less format, so you will need to make the schema relevant to the domain and relevant to the users. Consider using JSON Schema[^22] to formally define your data structures.

## Consistency and Documentation

I can't stress this enough: consistency coupled with good documentation will win over hearts and minds. The documentation should be indexed, searchable, include examples, and cover edge cases. The API Stylebook[^23] collects API design guidelines from various companies for reference.

---

## References

[^1]: Fielding, Roy Thomas. (2000). "Architectural Styles and the Design of Network-based Software Architectures." Doctoral dissertation, University of California, Irvine. https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
[^2]: Object Management Group. "Common Object Request Broker Architecture (CORBA) Specification." https://www.omg.org/spec/CORBA/
[^3]: EDI Basics. "What is EDI (Electronic Data Interchange)?" https://www.edibasics.com/what-is-edi/
[^4]: W3C. (2007). "SOAP Version 1.2 Part 1: Messaging Framework (Second Edition)." https://www.w3.org/TR/soap12/
[^5]: Martin, Robert C. "The Single Responsibility Principle." https://en.wikipedia.org/wiki/Single-responsibility_principle
[^6]: OpenAPI Initiative. "OpenAPI Specification." https://www.openapis.org/
[^7]: SmartBear. "Swagger UI." https://swagger.io/tools/swagger-ui/
[^8]: Redocly. "Redoc - OpenAPI/Swagger-generated API Reference Documentation." https://github.com/Redocly/redoc
[^9]: Stoplight. "API Design, Documentation, Mocking, and Testing." https://stoplight.io/
[^10]: OpenAPI Initiative. "OpenAPI Specification v3.1.0." https://spec.openapis.org/oas/latest.html
[^11]: SmartBear. "Swagger Codegen." https://swagger.io/tools/swagger-codegen/
[^12]: OpenAPI Generator. "OpenAPI Generator - Generate clients, servers, and documentation." https://openapi-generator.tech/
[^13]: Beyer, Betsy et al. (2016). "Site Reliability Engineering: How Google Runs Production Systems." O'Reilly Media. Chapter 6: Monitoring Distributed Systems. https://sre.google/sre-book/monitoring-distributed-systems/
[^14]: Uptime.is. "SLA Uptime Calculator." https://uptime.is/
[^15]: Wilde, E. (2019). "The Sunset HTTP Header Field." RFC 8594, IETF. https://datatracker.ietf.org/doc/html/rfc8594
[^16]: Grafana Labs. "k6 - A modern load testing tool." https://k6.io/
[^17]: Gatling Corp. "Gatling - Load Testing Tool." https://gatling.io/
[^18]: Locust. "An open source load testing tool." https://locust.io/
[^19]: Beyer, Betsy et al. (2016). "Site Reliability Engineering: How Google Runs Production Systems." O'Reilly Media. https://sre.google/sre-book/monitoring-distributed-systems/
[^20]: Principles of Chaos Engineering. "Chaos Engineering." https://principlesofchaos.org/
[^21]: Crockford, Douglas. "Introducing JSON." https://www.json.org/
[^22]: JSON Schema. "JSON Schema - The home of JSON Schema." https://json-schema.org/
[^23]: API Stylebook. "API Design Guidelines." http://apistylebook.com/

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Next: Pragmatism](02-pragmatism.md)
