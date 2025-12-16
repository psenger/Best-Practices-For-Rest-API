# Pragmatism

[Index](../README.md) | [Previous: The Human Aspect](01-human-aspect.md)

---

Being pragmatic means making decisions that work in the real world, not just in theory. The JavaScript and broader development ecosystem has a tendency toward over-engineering and dependency accumulation that can cripple long-term maintainability.

## Avoid Dependency Bloat

Every dependency you add is a liability. It's code you don't control, with its own bugs, security vulnerabilities, and breaking changes. Before adding a package, ask yourself:

1. **Can I write this myself in under 100 lines?** - If yes, consider doing so
2. **How many transitive dependencies does this bring?** - Check with `npm ls` or similar
3. **Is this package actively maintained?** - Check last commit date and open issues
4. **What's the download size impact?** - Use tools like Bundlephobia[^1]

The JavaScript ecosystem is particularly prone to "left-pad"[^2] situations where trivial functionality is packaged as dependencies. A package to check if a number is odd, a package to pad a string: these create fragility without adding real value. David Haney's analysis[^3] of the npm ecosystem's issues is worth reading.

**Practical guidelines:**

- Audit your `node_modules` regularly. Tools like npm-audit[^4] and Snyk[^5] can identify vulnerabilities.
- Prefer standard library functions over utility packages
- Be sceptical of packages with dozens of dependencies for simple tasks
- Consider vendoring small, stable utilities rather than depending on them
- Use Socket.dev[^6] to analyse supply chain risk

## Framework Lock-in

Adopting a framework is a marriage, not a date. Once your codebase is built around Express[^7], NestJS[^8], or any other framework, migrating away is expensive. Consider:

1. **Keep business logic framework-agnostic** - Your domain logic should not import Express types
2. **Use ports and adapters**[^9] - Abstract your HTTP layer from your business logic (also known as Hexagonal Architecture[^10])
3. **Prefer libraries over frameworks** - Libraries you call; frameworks call you. See Martin Fowler on Inversion of Control[^11].
4. **Evaluate longevity** - That hot new framework may be abandoned in two years

The more your code depends on framework-specific patterns, the harder it becomes to:

- Switch frameworks when better options emerge
- Test business logic in isolation
- Reuse logic across different contexts (CLI, serverless, etc.)

## Build vs Buy

Sometimes the pragmatic choice is to not build at all. Before implementing custom authentication, rate limiting, or API gateways, consider whether managed services or existing solutions fit your needs. But weigh this against:

1. **Vendor lock-in** - Can you migrate away if needed? See CNCF's guidance on avoiding lock-in[^12].
2. **Cost at scale** - Cheap at low volume doesn't mean cheap at high volume
3. **Complexity** - Does the "simple" solution actually simplify things?
4. **Control** - What happens when the service doesn't do exactly what you need?

The Thoughtworks Technology Radar[^13] is useful for evaluating technology choices and understanding industry trends.

## AI-Assisted API Development

AI tools like Claude[^14] can significantly accelerate API development when used pragmatically:

1. **Design review** - Have AI review your OpenAPI[^15] specs for consistency and best practices
2. **Boilerplate generation** - Generate repetitive CRUD handlers, validation schemas, and tests
3. **Documentation** - Generate endpoint descriptions and examples from code
4. **Refactoring** - Identify inconsistencies across endpoints and suggest standardisation

However, maintain human oversight. AI can generate plausible but incorrect code, especially around security, authentication, and edge cases. Use AI to accelerate, not to replace understanding. See OWASP's guidance on AI security risks[^16].

---

## References

[^1]: Bundlephobia. "Find the cost of adding a npm package to your bundle." https://bundlephobia.com/
[^2]: npm Blog. (2016). "kik, left-pad, and npm." https://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm
[^3]: Haney, David. (2016). "NPM & left-pad: Have We Forgotten How To Program?" https://www.davidhaney.io/npm-left-pad-have-we-forgotten-how-to-program/
[^4]: npm Documentation. "npm-audit." https://docs.npmjs.com/cli/v8/commands/npm-audit
[^5]: Snyk. "Developer Security Platform." https://snyk.io/
[^6]: Socket. "Secure your JavaScript supply chain." https://socket.dev/
[^7]: Express.js. "Fast, unopinionated, minimalist web framework for Node.js." https://expressjs.com/
[^8]: NestJS. "A progressive Node.js framework." https://nestjs.com/
[^9]: Cockburn, Alistair. "Hexagonal Architecture." https://alistair.cockburn.us/hexagonal-architecture/
[^10]: Wikipedia. "Hexagonal architecture (software)." https://en.wikipedia.org/wiki/Hexagonal_architecture_(software)
[^11]: Fowler, Martin. "Inversion of Control." https://martinfowler.com/bliki/InversionOfControl.html
[^12]: Cloud Native Computing Foundation. (2018). "6 Considerations for Avoiding Lock-in When Using Cloud Native Services." https://www.cncf.io/blog/2018/03/07/6-considerations-for-avoiding-lock-in-when-using-cloud-native-services/
[^13]: Thoughtworks. "Technology Radar." https://www.thoughtworks.com/radar
[^14]: Anthropic. "Claude - AI Assistant." https://www.anthropic.com/claude
[^15]: OpenAPI Initiative. "OpenAPI Specification." https://www.openapis.org/
[^16]: OWASP. "AI Security and Privacy Guide." https://owasp.org/www-project-ai-security-and-privacy-guide/

---

Written by [Philip A Senger](https://psenger.github.io/philip_a_senger_cv/) | [LinkedIn](https://www.linkedin.com/in/philipsenger/) | [GitHub](https://github.com/psenger/)

This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/).

[Previous: The Human Aspect](01-human-aspect.md) | [Next: Security and Permissions](03-security.md)
