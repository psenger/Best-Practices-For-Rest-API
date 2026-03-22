# Contributing

Thanks for your interest in contributing to **Best Practices for Building REST APIs**. This is a knowledge repository — contributions are prose, citations, and structured markdown, not code. The bar is accuracy, clarity, and real-world practicality.

---

## Before You Start — Raise an Issue First

**All contributions must begin with a GitHub issue.** This lets us discuss whether the change belongs here, agree on scope, and avoid wasted effort.

Open an issue using one of the templates:

- **Content Suggestion** — proposing a new topic, new chapter, or expansion of an existing section
- **Correction** — broken link, factual error, outdated reference, or typo

Wait for a response before writing. PRs opened without a linked issue may be closed without review.

---

## Workflow

1. **Open an issue** and get it acknowledged
2. **Fork** the repository
3. **Create a branch** from `master`:
   ```bash
   git checkout -b feat/issue-<number>-short-description
   ```
4. **Make your changes** (see writing guidelines below)
5. **Open a pull request** against `master` using the PR template

Keep PRs focused — one topic or one fix per PR.

---

## Writing Guidelines

### Tone

- Pragmatic and direct. This guide is opinionated by design.
- Written from experience, not from theory. If something doesn't work in practice, say so.
- Use "you" to address the reader. Use "we" sparingly.
- Avoid hedging language ("might", "could potentially", "in some cases") unless genuinely uncertain.

### Citations and Footnotes

Every factual claim, RFC reference, or external standard must have a footnote citation.

```markdown
Use TLS 1.3[^1] everywhere, no exceptions.

[^1]: Rescorla, E. (2018). "The Transport Layer Security (TLS) Protocol Version 1.3." RFC 8446, IETF. https://datatracker.ietf.org/doc/html/rfc8446
```

- Footnotes go at the bottom of the file under a `## References` heading.
- Number footnotes sequentially within each file, starting from `[^1]`.
- Use the full citation format: Author(s), year, title, source, URL.
- RFCs: include the RFC number, the IETF link, and the year.
- OWASP, NIST, W3C: link to the canonical source, not a blog summary.
- If adding a reference that belongs in `docs/11-sources.md`, add it there too.

### Markdown Conventions

- Use `##` for top-level sections within a chapter, `###` for subsections.
- Use tables for comparisons. Use fenced code blocks for examples (```` ``` ````).
- Use `>` blockquotes sparingly — only for genuine quotes or key principles worth calling out.
- GitHub callouts (`> [!TIP]`, `> [!NOTE]`, `> [!WARNING]`) are used intentionally — don't add them casually.
- Keep line length readable. No hard limit, but don't run paragraphs into walls of text.
- Navigation links at the top and bottom of each file follow the existing pattern:
  ```markdown
  [Index](../README.md) | [Previous: Chapter Name](prev-file.md)
  ```

### What Belongs Here

- Practical guidance backed by real-world experience or authoritative standards
- Trade-off analysis (not just "here's how", but "here's when and why")
- Patterns with named sources (Stripe's approach to X, Netflix's approach to Y)
- RFC or standards references when a behaviour is standardised

### What Doesn't Belong Here

- Framework-specific tutorials ("how to implement this in Express")
- Language-specific code beyond short illustrative snippets
- Opinions without grounding ("I think X is better")
- Content that belongs in API documentation rather than design guidance

---

## Adding a New Chapter

If your issue is approved for a new chapter:

1. Name the file `docs/NN-topic-name.md` where `NN` continues the sequence.
2. Follow the file structure of existing chapters:
   - Title as `#`
   - Navigation links (Index, Previous, Next)
   - `---` separators
   - Table of Contents if the chapter is long
   - `## References` section with footnotes
   - Author attribution and licence footer
3. Add the chapter to the `## Table of Contents` in `README.md`.
4. Add nav links to the preceding and following chapters.
5. Add any new sources to `docs/11-sources.md`.

---

## Licence

By contributing, you agree that your contributions will be licensed under the repository's [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/). You must be the original author of contributed content or have the right to contribute it under these terms.
