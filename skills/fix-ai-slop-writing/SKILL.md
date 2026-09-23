---
name: fix-ai-slop-writing
description: Turn a repository's AI-slop prose into a measured, lint-enforced plain technical register, rewrite the existing docs without losing a fact, and keep new AI writing clean. Use when the user complains about slop or AI-sounding writing, or wants a prose style guide, a prose linter, or a bulk rewrite of docs, READMEs, knowledge-base pages or UI copy.
---

# Fix AI slop writing

Slop is a measurement, not a taste. A corpus is sloppy when its numbers say so: mean sentence length near 30 words, half the sentences over 25, an em-dash every few hundred words, figurative verbs ("unlock", "leverage", "dive into"), empty intensifiers, Latin abbreviations, passive voice, two spellings of one word, three terms for one concept. This skill fixes it in four moves, and each move makes the next one safe:

1. A **style guide** whose rules are checkable, each with a real before and after.
2. A **linter** that enforces the guide, with a scope-by-scope **ratchet**.
3. A **retroactive rewrite** in small batched PRs, each gated by a **fact-preservation check**, so a rewrite changes words but never claims.
4. **Upkeep** that points every future writer, human or agent, at the guide and the lint.

The target register is Simplified Technical English (STE, after ASD-STE100) used as a base, not as the strict standard: take its principles (short sentences, one idea per sentence, active voice, literal verbs, one term per concept) and skip its controlled dictionary. The project's own domain vocabulary stays, and the style guide adapts the principles to the project.

A rule no tool checks will rot, and a convention that lives only in prose gets lost in a merge. So wherever this skill says "convention", it means a rule in the guide **and** a check in the lint.

## Steps

### 1. Measure the baseline

Build the metrics script before any rule. For every prose unit in scope, count: words, sentences, mean words per sentence, share of sentences over 25 and over 30 words, longest sentence, em-dashes (total and per 1k words), spaced hyphens used as dashes, Latin abbreviations, figurative words, empty intensifiers, passive constructions (heuristic), off-convention spellings, repeated connectives, repeated sentence openers. Emit one row per scope (a directory or file type) and the longest sentences.

Done when: a generated table regenerates from one command, and you can quote its headline numbers to the user.

### 2. Write the style guide

Follow [`references/style-guide.md`](references/style-guide.md). The guide lists which text is in scope, marks every rule as lint-enforced or reviewer-checked, gives every rule a real before and after with a revision-pinned location, and states what text is never changed.

Done when: every rule names its tier and its check, every example is quoted from the repository, the never-change list covers quoted text, code, numbers, identifiers, links, hedges and test-pinned strings, and the rules section fits in about 200 lines.

### 3. Build and dogfood the tooling

Follow [`references/tooling.md`](references/tooling.md). Ship it as the first PR of the series. It adds one shared text library, the linter, the mechanical fixer, the reflow tool, the hash-guarded edit applier and the fact-preservation check. It rewrites no prose and enables no scope, so CI stays green and the lint output is the baseline table.

Before the PR merges, **dogfood**: run every tool over the whole corpus and over the style guide itself. Turn each false positive into a regression test and fix the mask. The guide passes its own lint.

Done when: `prose:check` prints the per-scope, per-rule table; `rewrite:check` against the default branch reports zero changed files; the round-trip test proves that one spelling of every notation construct passes both the linter and the fact check; the guide has zero fail-tier hits.

### 4. Calibrate, then fold the results into the tooling

Tag the tree before the first rewrite and publish the tag (`git tag pre-rewrite && git push origin pre-rewrite`). Every later gate compares against it, so CI must fetch tags (`git fetch --tags`, or `fetch-depth: 0` with `fetch-tags: true` in GitHub Actions). Record the tagged commit hash in the lint config too, so a clone without tags can still resolve the baseline. Rewrite one small batch (about 10 files or sections) per content type, following [`references/batches.md`](references/batches.md). A human reads each calibration batch end to end.

Calibration is where most of the quality gain happens. Turn every judgment call into a guide rule plus a lint check or word-list entry, merge that into the tooling branch, and only then start the bulk batches from the updated tooling.

Done when: every calibration convention exists as a guide rule and a check on the tooling branch, and the calibration PRs list them.

### 5. Fan out the batches

Cut the rest of the corpus into batches of about 25 files or sections, one PR each, with the same commit layers and gates. Keep the tooling branch live: when a tool blocks a legitimate edit, fix the tool, merge the fix, and rebase the open batches instead of deferring the fix to the end. When a scope reaches zero fail-tier hits, the PR that finishes it adds the scope to the lint config's `enabled` list.

Done when: every batch PR is merged in manifest order with a check report showing zero lost facts and its metrics row before and after.

### 6. Close the series

A final PR regenerates derived files once, runs the fact check over the whole corpus against the `pre-rewrite` tag, re-runs the mechanical fixer everywhere, diffs the style guide against the tooling branch tip to catch rules a merge dropped, enables every clean scope, and adds an "Open items" section to the guide that says what was rewritten and what was deferred.

Done when: the whole-corpus check reports zero failures, the guide matches the tooling tip, and the open items name every disabled scope with its remaining hit count.

### 7. Keep it clean

Follow [`references/upkeep.md`](references/upkeep.md): an `AGENTS.md` pointer, links instead of copied rules in every skill that writes prose, a Markdown scope, a diff-only lint mode, and a hook.

Done when: an agent that has never seen the guide finds it from `AGENTS.md`, and a new em-dash in any Markdown file fails CI.

## Pitfalls

Read [`references/pitfalls.md`](references/pitfalls.md) before step 3 and again before step 5. It lists what went wrong in practice and what to expect of the results.
