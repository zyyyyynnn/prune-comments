# prune-comments

Preserve contracts while pruning comment noise. The skill classifies every item, fixes root causes behind workarounds, and reports unresolved claims as `open`.

## When to reach for it

Reach for `prune-comments` when the task is:

- Removing redundant, outdated, or AI-generated comment noise.
- Auditing lint and type suppressions.
- Moving durable decisions into existing project docs.

For general code review, code explanation, or public API writing, use a purpose-built skill.

## Installation

### Claude Code plugin

```bash
claude plugin marketplace add zyyyyynnn/prune-comments
claude plugin install prune-comments@zyyyyynnn
```

### Vercel Skills

Install with [Vercel Skills](https://github.com/vercel-labs/skills):

```bash
# Project-local
npx skills add zyyyyynnn/prune-comments

# Global
npx skills add -g zyyyyynnn/prune-comments
```

Or copy `skills/prune-comments/` into your agent's skill directory:

- Cursor: `.cursor/skills/prune-comments/`
- Claude Code: `.claude/skills/prune-comments/`
- Codex / Gemini CLI / Universal: `.agents/skills/prune-comments/`

## What it does

The skill treats a comment as a **contract** when deleting it would change a public API, external constraint, executable directive, safety invariant, or decision link. Contracts are kept only with evidence.

Every item receives a stable identity and one outcome: `deleted`, `kept`, or `open`. Root-cause fixes are listed separately from comment-only edits. Large scopes use resumable review shards, and a partial review releases only independently reviewed rows.

## Example

```ts
// === Helpers ===
/** Returns whether the user account is active. */
export function isActive(u: { status?: string }): boolean {
  // Check if user is active (returns true if active)
  // @ts-ignore - legacy payload sometimes omits status, works for now
  return u.status.toLowerCase() === "active";
}
```

After cleanup:

```ts
/** Returns whether the user account is active. */
export function isActive(u: { status?: string }): boolean {
  return u.status?.toLowerCase() === "active";
}
```

The banner and redundant comment are deleted, the suppression is replaced with a nullability fix, and the public API docstring is preserved.

## It's working if

- Every item ends as `deleted`, `kept`, or `open`, and the totals reconcile.
- No proven contract is deleted without a structural replacement.
- Root-cause fixes are listed separately from comment-only edits.
- Unresolved claims name an owner and next step.
- Configured checks pass, and unrun checks are explicit.

## Usage

Invoke with natural language or host shortcuts (`/prune-comments:prune-comments` for the Claude Code plugin, `/prune-comments` for manual Claude Code/Cursor installs, `$prune-comments` in Codex):

- `Audit comments in src/auth/ with prune-comments; report only, without editing.`
- `Run prune-comments on my current branch diff.`
- `Use prune-comments on src/payments/; reuse our existing engineering docs.`

## Structure

```text
.
├── .claude-plugin/
├── .github/
├── CONTRIBUTING.md
├── SECURITY.md
├── README.md
└── skills/prune-comments/
    ├── SKILL.md
    ├── agents/openai.yaml
    └── references/
        ├── audit-protocol.md
        └── language-rules.md
```

The entry file holds the workflow. `audit-protocol.md` defines the inventory used by every run; `language-rules.md` loads when syntax-specific judgment is needed.

## License

[MIT](LICENSE)
