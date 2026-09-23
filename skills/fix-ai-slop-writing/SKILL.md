---
name: fix-ai-slop-writing
description: Turn a repository's AI-slop prose into a measured, lint-enforced plain technical register, then rewrite the existing docs without losing a fact. Use when the user complains about slop or AI-sounding writing, or wants a prose style guide, a prose linter, or a bulk rewrite of docs, READMEs, knowledge-base pages or UI copy.
---

# Fix AI slop writing

Slop is a measurement, not a taste. A corpus is sloppy when its numbers say so: mean sentence length near 30 words, half the sentences over 25, an em-dash every few hundred words, figurative verbs ("unlock", "leverage", "dive into"), empty intensifiers, Latin abbreviations, passive voice, two spellings of one word, three terms for one concept. This skill fixes it in three moves, and each move makes the next one safe:

1. A **style guide** whose rules are checkable, each with a real before and after.
2. A **linter** that enforces the guide, with a scope-by-scope **ratchet**.
3. A **retroactive rewrite** in small batched PRs, each gated by a **fact-preservation check**, so a rewrite changes words but never claims.

The target register is Simplified Technical English (STE, after ASD-STE100): short sentences, one idea per sentence, active voice, literal verbs, one term per concept.

## Steps

### 1. Measure the baseline

Build the metrics script before any rule. For every prose unit in scope, count: words, sentences, mean words per sentence, share of sentences over 25 and over 30 words, longest sentence, em-dashes (total and per 1k words), spaced hyphens used as dashes, Latin abbreviations, figurative words, empty intensifiers, passive constructions (heuristic), off-convention spellings, parentheses per 1k. Emit one row per scope (a directory or file type) and the longest sentences.

Done when: a generated table (for example `docs/prose-metrics.md`) regenerates from one command, and you can quote its headline numbers to the user.

### 2. Write the style guide

Follow [`references/style-guide.md`](references/style-guide.md). The guide lists which text is in scope, splits rules into a mechanical tier and a review tier, gives every rule a real before and after with `file:line`, and states what text is never changed.

Done when: every rule names its tier and its check, every example is quoted from the repository, and the never-change list covers quoted text, code, numbers, identifiers, links, hedges and test-pinned strings.

### 3. Build the linter and the rewrite tooling

Follow [`references/tooling.md`](references/tooling.md). Ship it as the first PR of the series. It adds the linter, the mechanical fixer, the hash-guarded edit applier and the fact-preservation check. It rewrites no prose and enables no scope, so CI stays green and the lint output is the baseline table.

Done when: `prose:check` prints the per-scope, per-rule table; `rewrite:check` against the default branch reports zero changed files; every masking edge case (quotes, code, links, math) has a unit test.

### 4. Calibrate on one small batch per content type

Tag the tree before the first rewrite (`git tag pre-rewrite`). Rewrite one small batch (10 to 50 files or sections) per content type, following [`references/batches.md`](references/batches.md). A human reads each calibration batch end to end. Every judgment call the reviewers make becomes a written convention for the remaining batches, or a one-row change to the style guide on the tooling branch.

Done when: each calibration PR lists the conventions it established and the style-guide gaps it deferred to the tooling branch.

### 5. Fan out the batches

Cut the rest of the corpus into batches of about 25 files or sections, one PR each, with the same commit layers and gates. Batches run in parallel; each branches from the tooling branch. When a scope reaches zero fail-tier hits, the PR that finishes it adds the scope to the lint config's `enabled` list, and CI enforces that scope from then on.

Done when: every batch PR is merged with a check report showing zero lost facts and its metrics row before and after.

### 6. Close the series

A final PR regenerates derived files (squash merges in a different order leave them stale), runs the fact check over the whole corpus against the `pre-rewrite` tag, enables every clean scope, and adds an "Open items" section to the style guide that says what was rewritten and what was deferred, with the remaining hit counts per disabled scope.

Done when: the whole-corpus check reports zero failures and the open items name every disabled scope and the reason.

## Pitfalls

Read [`references/pitfalls.md`](references/pitfalls.md) before step 3. It lists the defects this process tends to let through, the conventions that calibration usually produces and what to expect of the results.
