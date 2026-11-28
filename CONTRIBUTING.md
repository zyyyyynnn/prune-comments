# Contributing

This repository ships one agent skill. Keep changes small, language-independent, and focused on the skill's job: auditing and pruning comments without deleting contracts.

## Issues and pull requests

- Report one problem per issue. Put the smallest reproduction before analysis.
- Use the bug template for incorrect behavior and the idea template for proposed changes.
- Open a pull request for concrete changes. Include the problem, the intended behavior, and the verification you ran.
- Do not include credentials, private session logs, or customer data.

## Skill changes

- Keep `SKILL.md` as the short workflow entry.
- Put branch-specific detail in `references/` behind a clear pointer.
- Preserve cross-project and cross-language applicability. Project-specific rules belong in examples or optional reference material.
- Update `README.md` when user-visible behavior changes.
- Keep the five `Sole Exceptions` authoritative in `SKILL.md`.

## Before submitting

- Confirm valid frontmatter and working relative links.
- Confirm no unresolved placeholders or project-specific residue.
- Run `git diff --check`.
- Exercise the changed branch in a scratch project or dry-run when practical, and report what you ran.
