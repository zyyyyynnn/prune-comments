# Language Rules

Use this reference when syntax alone cannot establish whether a comment is a contract, directive, or removable prose. Project conventions and active tooling override defaults; record the evidence used.

## General Resolution

For every doc comment or suppression, resolve:

1. **Effective visibility**: declaration visibility, module path, re-exports, package exports, and conditional compilation.
2. **Runtime consumers**: macros, CLI help generators, schema/doctest tools, serializers, and documentation generators.
3. **Tool requirements**: compiler, linter, formatter, test, coverage, and build rules active for the current target and features.
4. **Association**: the exact symbol, expression, unsafe operation, attribute, or test the comment governs.
5. **Project policy**: zero-suppression rules, private-item documentation rules, and generated-file policies.

Do not equate `public-looking` syntax with an external API, and do not exempt a comment solely because its prefix looks directive-like.

## Rust

- Resolve effective visibility through exports, re-exports, `cfg`, and macros. `pub` alone does not prove an external API; binary, private, and test-only items follow project convention.
- Treat docs consumed by CLI/schema/doctest tools or macro expansion as contracts only when that consumer is active.
- Keep `# Safety` docs when an active tool or project policy requires them. Keep `SAFETY:` block directives only when tied to a real unsafe operation; an orphan directive on safe code is not exempt.
- Detect `allow`, `expect`, conditional forms, and formatter directives with a parser/tokenizer, including whitespace and nested attributes. Zero-suppression policy remains project-specific.

## JavaScript and TypeScript

- Resolve package exports, barrel re-exports, declaration visibility, and runtime decorators before treating JSDoc/TSDoc as public API.
- Preserve executable type-check directives such as `@ts-check`, `/// <reference>`, and justified `@ts-expect-error` in type tests.
- Classify `eslint-disable`, `oxlint-disable`, and `biome-ignore` by the suppressed rule and project policy.
- Use a parser/scanner that distinguishes comments from strings, template literals, regex literals, URLs, and JSX text.

## Python

- Resolve module exports, `__all__`, package API conventions, decorators, and runtime consumers.
- Preserve executable doctests, encoding cookies, shebangs, type-check directives, and tool-required safety/contract docs.
- Classify `# type: ignore`, `# noqa`, and similar suppressions by rule and active configuration.
- Use `tokenize` for comments and `ast` for docstrings, decorators, and symbol association.

## Go

- Exported identifiers may be public API by language convention; confirm module and package boundaries.
- Preserve build tags, `cgo` preambles, `// Output:` / `// Unordered output:` examples, and tool-required directives.
- Classify `//nolint` by rule and project policy.
- Use `go/parser` or equivalent; do not infer comment boundaries from raw lines.

## Deletion Pitfalls

- **Python**: Replacing a sole-statement docstring requires `...` or `pass`; otherwise the body becomes invalid.
- **JSX**: Remove the entire `{/* ... */}` container, not only its inner comment text.
- **Go/cgo**: Preserve adjacency between cgo preamble comments and `import "C"`.

## Extraction Fixtures

When applicable to the language and toolchain, test:

- Comment markers and suppression text inside strings, raw strings, templates, regexes, and macro input.
- Nested block comments and multi-line attributes.
- URLs, hex colors, and documentation links.
- Valid and orphan safety comments.
- Public, private, re-exported, test-only, and conditionally compiled symbols.
- Included or generated docs and target/feature-specific code.
