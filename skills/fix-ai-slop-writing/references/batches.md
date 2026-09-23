# Batch PRs

## Series shape

1. **Tooling PR**: style guide, shared library, linter, fixer, reflow, applier, fact check, metrics. No prose changes, no scope enabled.
2. **Cross-cutting PRs**: renames and structural moves many batches depend on (a product name, a glossary term, a page template, the UI copy).
3. **Restructure PRs**, one per content type whose layout changes. Mechanical restructure first, hand rewrite second.
4. **Calibration batches**, about 10 files each, one per content type. Their conventions merge into the tooling branch before step 5.
5. **Content batches** of about 25 files or sections each. Findings per batch do not grow with batch size, so 25 is safe once calibration is done.
6. **Final PR** (see SKILL.md step 6).

Write a **manifest** first: every batch, its files, and its merge order. Every batch branches from the current tooling tip. Title them `Rewrite <scope> in plain English, batch k of N`, and mark calibration batches.

## Commit layers, in order, never mixed

1. **Reflow.** One sentence per line. Whitespace only.
2. **Mechanical.** `prose:fix` for the mechanical tier. Reviewed line by line.
3. **Rewrite.** The model applies the review tier, the glossary and the never-change list through edit files, runs `rewrite:check --edits`, then `rewrite:apply`, about five files per commit.
4. **Review fixes**, through the same `rewrite:apply` path.

Each layer's diff answers one question. Batch PRs change prose only: no shared script, no word-list entry and no generated file. A tooling change goes to the tooling branch and the batch rebases onto it.

**Generated files.** Do not commit regenerated views, metrics or appendices in batch PRs. Parallel batches that each regenerate them conflict, go stale when merges land out of order, and fill the review bot's output with "stale file" findings. Regenerate once in the final PR, and configure the review bot to ignore generated paths.

## The live tooling loop

When the fact check or the lint blocks an edit a reviewer agrees with, do not work around it in prose. Open a small tooling PR with the failing example as a test, merge it, rebase the open batches, and re-run their gates. Deferring tool fixes to the end leaves every later batch fighting the same bug and pushes the fixes into the final PR.

## Agents per batch

- **Rewriter.** Gets the style guide, the never-change list, the calibration conventions, the batch's files and their unit hashes. Applies every rule to every field, including titles and names. Writes edit files and runs `rewrite:check --edits` until it passes.
- **Fidelity verifier.** Reads five sampled files end to end against the `pre-rewrite` tag, then runs its own token diff over the whole batch. Its checklist, since token counts miss all of these:
  - a hedge, condition or attribution removed, weakened or strengthened;
  - a gloss, actor, cause or claim the original did not state;
  - a general claim narrowed to one case, or one case widened to a general claim;
  - two cases merged, or a condition that did not survive a sentence split.
- **Prose reviewer.** Reads every file for the register: garden-path sentences, clipped sentences and verbless stubs, fragments where the original had a full sentence, repeated openers, connective chains, arrow chains in place of sentences. A style claim cites a measurement ("the sentence is 34 words with citations masked", "four consecutive sentences open with 'The request'"). An unmeasured claim is often wrong.
- **Integrator.** Applies accepted findings and rejects the rest with a reason.

Tell both reviewers: **files not yet rewritten are not a standard to match.** Halfway through a series, the old text looks like a competing convention.

## Rejection rules

- A finding that contradicts the guide loses inside the batch. If the finding is right, change the guide on the tooling branch, never add a per-file exception.
- A finding blocked by a tool goes through the live tooling loop.
- A content error that predates the rewrite is out of scope for a style pass; log it as an issue.
- A suggested wording that adds a fact is rejected.

## Gates on the last commit

Build, unit tests, `prose:check` (no fail-tier hit in an enabled scope, and zero in the batch's files), `prose:fix --check` and `prose:reflow --check` on the batch's files, end-to-end UI tests if the batch touches UI copy, and `rewrite:check --base pre-rewrite <batch paths>` with zero failures.

## PR description

Generate it from the manifest and the check report, not from the previous batch's description: a copied description carries another batch's findings. It holds:

1. The batch's files, from the manifest.
2. The fact-check summary: files, units, tokens checked, failures, and each warning explained.
3. Findings fixed, one clause each, naming the file.
4. Findings rejected, each with its rule, and tooling PRs opened.
5. Gates run and their results.
6. How to review one file: `git diff pre-rewrite -- <path>`.

A calibration PR adds: "Conventions for the remaining batches", each with the guide rule and the check it became.

## Merging

Merge in manifest order. After each merge, rebase the next batch and re-run its gates. A batch that squash-merges from a stale tooling base can silently revert guide rules added since it branched.

## Final PR checklist

- Regenerate every generated file once.
- Run `rewrite:check --base pre-rewrite` over the whole corpus. Per-batch checks pass while cross-batch interactions fail; restore the original wording where they do.
- Run `prose:fix --check` and `prose:reflow --check` over the whole corpus to catch files that skipped a layer.
- Diff the style guide and the word list against the tooling branch tip to catch rules a merge dropped.
- Enable every clean scope.
- A before-and-after metrics table, one row per scope.
- The guide's "Open items": done, deferred, and remaining hit counts per disabled scope.
