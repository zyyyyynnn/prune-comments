---
name: prune-comments
description: "Audit and prune redundant, outdated, or workaround-excusing comments, commented-out code, and lint/type suppressions across a diff, file, or directory. Use for comment cleanup, AI-slop comment removal, or lint-ignore audits. Supports resumable multi-shard review; not for general code review, code explanation, or API documentation."
license: MIT
compatibility: Requires git for diff and history inspection and a language parser/tokenizer for inventory.
metadata:
  version: "1.2.0"
---

# Prune Comments

Prune prose while preserving **contracts**. Classify every item, fix the root cause behind workarounds, and leave uncertainty `open`.

A **contract** is a comment whose removal changes a public API, external constraint, executable directive, safety invariant, or decision link.

## Choose a mode

| Mode | Run |
| --- | --- |
| Dry-run | Steps 1-2. Report proposed actions without edits or tests. Independent review is optional; state whether it ran. |
| Cleanup (default) | Steps 1-4. |

## Scope

Use the caller's files, directory, or diff. Otherwise audit `git diff <default-branch>...HEAD` plus staged, unstaged, and untracked changes. Resolve `<default-branch>` from `origin/HEAD`, then `main`, then `master`; ask for a base or path if none resolves. If `HEAD` is unborn, audit the working tree only.

- **Diff scope**: Audit changed lines and adjacent comments that describe changed symbols.
- **Explicit scope**: Confirm the path exists and contains auditable source files. Git history is optional outside a repository.
- **Skip**: Vendored/build directories, lockfiles, and files marked generated.
- **Inventory**: Use parser/tokenizer output as the source of truth. Apply the schema and identity rules in [`references/audit-protocol.md`](references/audit-protocol.md).

Stop and report when the resolved scope is empty.

## Independent review

Before cleanup edits, dispatch one fresh-context, read-only reviewer to repeat Steps 1-2 independently.

Give the reviewer only:

- Resolved scope and auto-skip rules.
- `Sole Exceptions`, Steps 1-2, `Gotchas`, and the report schema.
- The raw inventory, without prior classifications or desired deletions.

Capture a pre-review snapshot that can isolate reviewer edits: `HEAD`, staged and unstaged diffs, and untracked contents in Git; copies of in-scope files otherwise. The reviewer may write only its assigned scratch inventory.

If sub-agents are unavailable, disclose that and get explicit approval for a same-context audit. In non-interactive edit mode, stop and report review unavailable. Dry-run may proceed with review marked `not performed`.

### Acceptance gate

The main agent owns every accepted finding.

- Inspect the report and working-tree diff.
- Reject reviewer edits, scope escapes, unsupported flags, and exception-protected deletions.
- Require an exact symbol and root-cause action for every `MUST FIX`.
- Recheck high-signal claims such as `IMPORTANT`, `do not remove`, `too risky`, and `fine for now` against live code and history.
- Treat correctness or safety suppressions as `MUST FIX` unless a `Sole Exception` is proven.
- Classify unproven claims as `open`.
- For `accepted-partial`, edit only independently reviewed rows; gaps remain `open` and untouched.

On rejection, revert only confirmed reviewer-introduced edits and rerun once with a fresh reviewer, naming the failure. A second rejection stops the run before remediation. Use the coverage and status definitions in [`references/audit-protocol.md`](references/audit-protocol.md).

## Sole Exceptions

Use these categories to justify `kept`. Unresolved items remain `open`, even when a minimal tracking line stays in code:

1. **Legal or license header**: Required copyright, license, or SPDX notice.
2. **Public API contract doc**: Project-recognized interface, SDK, symbol, doctest, or schema documentation. Resolve effective visibility, re-exports, conditional compilation, and runtime consumers. See [`references/language-rules.md`](references/language-rules.md).
3. **Verified external constraint**: Non-obvious behavior forced by a dependency, platform, vendor, or protocol that cannot be reshaped and is active on a live path.
4. **Executable directive or invariant**: Compiler, runtime, build, formatter, safety, test, coverage, or type-test directive tied to the operation and active tool rule it governs. A prefix alone is not proof. Lint suppressions qualify only when the rule is faulty or purely stylistic; correctness and safety suppressions remain `MUST FIX`. Project-wide zero-suppression policy remains local.
5. **Minimal constraint link**: One line linking an existing decision document, RFC, or external spec for a constraint code cannot express. Validate the target and anchor.

## Workflow

Give every item exactly one post-remediation `outcome`. Compute totals from outcomes, not proposed actions.

| Outcome | Meaning |
| --- | --- |
| `deleted` | Removed completely, including a migrated decision with no leftover comment. |
| `kept` | Proven `Sole Exception`, or a migrated decision reduced to one valid decision-doc link. |
| `open` | Unresolved and still present, pending approval, evidence, or out-of-scope work. |

The accounting invariant is `total == deleted + kept + open`.

### 1. Inventory and verify

1. Build the deterministic inventory from [`references/audit-protocol.md`](references/audit-protocol.md), covering comments, commented-out code, and language-native suppressions.
2. Trace high-signal claims and long justifications through surrounding code, referenced symbols, dependency sources, and Git history.

*Done when*: every item has a stable identity, line span, and verified status.

### 2. Classify

Map each non-exempt item to one category:

- **Redundant**: Restates visible code, banners, or dead code.
- **Outdated**: Describes removed code, obsolete behavior, history, or process logs.
- **Excuse**: Defends a workaround, shortcut, or suppressed correctness/safety check.
- **Decision record**: Captures a durable architectural trade-off or domain rule.
- **Claimed constraint**: Asserts an invariant such as ordering, ownership, or wording.

*Done when*: every item maps to one `Sole Exception` or one non-exempt category.

### 3. Remediate

Batch approvals for public API or behavior changes, new decision docs, and structural checks into one prompt. Sketch non-trivial structural fixes before editing: exact symbol, root cause, proposed shape, and scope boundary.

| Change type | Rule |
| --- | --- |
| `comment-only` | Text change with no compiler, linter, formatter, test, runtime, schema, CLI-help, or doc-generation effect. |
| `code-change` | Any source or tool-consumed comment change. List it separately, preserve behavior and error taxonomy unless approved, and add focused regression tests. |

A `code-change` that touches unsafe code, authentication, authorization, concurrency, or protocols needs a second independent review or maintainer approval.

- **Redundant / Outdated**: Delete.
- **Excuse**: Delete the prose and apply the smallest in-scope root-cause fix. Keep out-of-scope workarounds as minimal tracking lines under `open`.
- **Decision record**: Move current facts and rationale into an existing architecture, engineering, or product document. Ask before creating `docs/adr/` when no suitable doc exists. Keep one decision-doc link only when code cannot self-explain, and validate its anchor.
- **Claimed constraint**: Prefer structural enforcement in this order: type system, runtime assertion, unit test. Get approval for new checks. Delete the comment only after the constraint is encoded; otherwise keep a minimal link under `open`.

In non-interactive runs, keep public API changes, new runtime assertions, and new decision docs under `open`.

*Done when*: edits stay inside scope plus approved decision docs and tests, and no verified invariant is deleted without replacement.

### 4. Verify and report

Run configured formatters, linters, type checks, and tests. Record exact commands, exit codes, and observed counts. Mark missing checks `not configured` and unrun checks with a reason.

```markdown
## Prune Comments Report

- **Scope / Base**: `<files or ref..HEAD>` / `<base>`
- **Unit**: `audit-item` (`comment-block` or `suppression`)
- **Review**: `<status per audit-protocol>` (`<reruns>`)
- **Reviewers / Shards**: `<counts or ids>`
- **Coverage / Skipped**: `<files and items covered>` / `<paths and reasons>`
- **Inventory**: `<total>` scanned (`<deleted>` deleted, `<kept>` kept, `<open>` open; `<comment_blocks>` blocks, `<suppressions>` suppressions)
- **Deleted**: `<N>` (`<redundant>`, `<outdated>`, `<excuse>`, `<decision>`, `<constraint>`)
- **Kept**: `<file:line: exception clause>`
- **Docs**: `<path#section: migrated decision>`
- **Fixes**: `<file:symbol: comment-only | code-change: root cause and evidence>`
- **Open**: `<file:line: claim, owner, next step>`
- **Verification**: `<command> -> exit <code> (<summary>)`
- **Not run**: `<check and reason>`
- **Artifacts**: `<inventory, shard, and session index paths>`
```

*Done when*: the accounting invariant holds, the review gate is satisfied, and all configured checks pass.

## Gotchas

- **Visibility**: Resolve re-exports, conditional compilation, macros, and runtime consumers before treating docs as public contracts.
- **Tool-consumed docs**: Verify active CLI-help, schema, doctest, safety, and documentation tooling.
- **Refactor fragments**: Look for split doc blocks, duplicate blocks, dangling sentences, and comments attached to the wrong symbol.
- **Dependency drift**: Check manifests, lockfile resolution, and live dependency sources.
- **Scanner boundaries**: Exercise strings, raw strings, nested comments, macros, attributes, URLs, and color literals when supported.
- **Docs and EOL**: Validate migrated anchors. After mass edits, run the repository's EOL check and `git diff --check`.
- **Language pitfalls**: Read [`references/language-rules.md`](references/language-rules.md) for syntax-sensitive deletion hazards and directive forms.
