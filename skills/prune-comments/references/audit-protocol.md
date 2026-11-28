# Audit Inventory, Sharding, and Recovery

Use the Inventory and Review Status sections for every audit. Apply Sharding, Recovery, and Merge Validation when the scope spans multiple reviewers or exceeds one reviewer output. Parser and visibility details live in `language-rules.md`.

## Inventory

Generate the inventory before classification with a parser/tokenizer. Do not infer counts from raw `grep`, regex matches over source text, or reviewer memory.

One audit item is either:

- A contiguous comment block. For line comments, use the maximal run of same-kind lines with no intervening code. Treat each block comment as one item regardless of line count.
- One lint/type suppression directive, including multi-line or conditional forms.

A directive comment counts only as a suppression and is excluded from comment-block counts, even though it uses comment syntax. When prose and an executable directive share one block, classify the block as `suppression` and record the prose in its evidence.

Use a deterministic, persisted schema:

```text
id | file | start | end | kind | text_preview | text_hash | visibility | test_scope | classification | action | evidence | outcome
```

- `kind`: `line`, `doc`, `block`, `suppression`, or a language-specific equivalent normalized by the generator.
- `id`: globally unique and stable for the run. Persist the generator's ordering; do not renumber after review starts.
- `text_hash`: hash normalized comment text or directive text. Use it to remap line drift; never silently accept changed text under an old ID.
- `visibility` and `test_scope`: record resolved facts, not guesses. Use `unknown` when unresolved.
- Reviewers fill only `classification`, `action`, and `evidence`. The main agent owns coordinates, IDs, merges, and totals, and records `outcome` (`deleted`, `kept`, or `open`) only after remediation.

Small scopes may keep the same schema in memory. Persist it to an approved scratch or repository artifact when recovery, multiple shards, or later verification is needed.

## Sharding

Partition inventory IDs into disjoint shards using the host's context and output budget. File and item counts are heuristics only; module boundaries and fixed thresholds are not correctness rules.

Each reviewer receives:

- Its exact ID range and file scope.
- The current `Sole Exceptions`, Steps 1-2, `Gotchas`, and report schema.
- The inventory rows needed for judgment, without prior classifications or desired deletions.

Reviewers are read-only with respect to the repository. They may write only their assigned scratch inventory. Persist completed rows after each file or small batch. If scratch writes are unavailable, stream bounded chunks and have the main agent persist them before requesting more.

Each shard checkpoint records:

```text
shard_id | scope_files | files_read | files_skipped | persisted_ids | missing_ids | process_state
```

`process_state` is `running`, `quota-blocked`, `context-limited`, or `complete`. Quota errors, turn limits, and context compaction are process states, not classification failures.

## Recovery

Resume from the persisted ID set: `missing_ids = assigned_ids - persisted_ids`. Never infer completeness from `last_id` or row count, because writes may complete out of order. Give only the missing IDs to a fresh reviewer with the original rules and raw inventory, without prior classifications. Never resend the entire completed range merely because the final message was lost.

Keep parallelism within the host's available budget.

## Merge Validation

Before accepting any completed shard, the main agent verifies:

1. Returned IDs equal that shard's assigned IDs, with no duplicates or omissions.
2. File, span, kind, and text hash match the source inventory, or drift is explicitly remapped.
3. `files_covered ∪ files_skipped == scope_files`, with a reason for every skip.
4. Every action belongs to `delete`, `fix`, `migrate to doc`, `encode`, `keep exempt`, or `open`.
5. Every `fix` or `encode` names a `MUST FIX` symbol and root-cause action.
6. Before remediation, compute proposed-action counts for planning only; do not treat them as final `deleted` / `kept` / `open` totals.

A reviewer checkpoint is evidence, not a substitute for the inventory. Reject incomplete merges rather than inferring missing classifications.

Apply `accepted-partial` only after all completed shards pass merge validation. Overall gaps are listed as `open` and unedited; an incomplete shard is a failed shard, not an `accepted-partial` result.

After remediation, require every row to have exactly one `outcome` (`deleted`, `kept`, or `open`) and compute `total == deleted + kept + open` from those outcomes.

## Review Status

- `accepted`: 100% of scope has reviewed rows and the main agent passed the acceptance gate.
- `accepted-partial`: the reviewer covered only part of the scope. Gaps are listed, remain `open`, and must not be edited; only independently reviewed rows may be remediated. Caller approval cannot convert an unreviewed gap into an approved edit.
- `fallback-approved`: no sub-agent was available and the caller approved a same-context audit.
- `not performed`: dry-run without independent review.
- `failed`: no usable review and no approved completion; stop before edits.

## Artifacts and Metrics

Retain inventory, shard, merge, and session-index paths in the report. When useful, record inventory time, coverage, recovery count, dropped IDs, merge failures, and high-signal recheck rate. Metrics diagnose the process; they never replace row-level accounting.
