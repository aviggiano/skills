# Batch PRs

## Series shape

1. **Tooling PR**: style guide, linter, fixer, reflow, applier, fact check, metrics. No prose changes, no scope enabled.
2. **Cross-cutting PRs**: renames and structural moves many batches depend on (a product name, a glossary term, a page template, the UI copy).
3. **Restructure PRs**, one per content type whose layout changes (for example reference pages into a fixed heading template). Mechanical restructure first, hand rewrite second.
4. **Calibration batches**, one per content type.
5. **Content batches** of about 25 files or sections each.
6. **Final PR** (see SKILL.md step 6).

Every batch branches from the tooling branch. Title them `Rewrite <scope> in plain English, batch k of N`, and mark calibration batches.

## Commit layers, in order, never mixed

1. **Reflow.** One sentence per line. No word changes.
2. **Mechanical.** `prose:fix` for the mechanical tier. Reviewed line by line.
3. **Rewrite.** The model applies the review tier, the glossary and the never-change list through edit files and `rewrite:apply`, four or five files per commit.
4. **Generated files**, regenerated in the same commit as the text they derive from.
5. **Review fixes**, through the same `rewrite:apply` path.

Each layer's diff answers one question. The reflow diff is word-identical, the mechanical diff is a script's output, and only layer 3 needs judgment.

## Agents per batch

- **Rewriter.** Gets the style guide, the never-change list, the batch's files and their unit hashes. Writes edit files, runs `rewrite:check --edits` until it passes, then applies.
- **Fidelity verifier.** Reads five sampled files end to end against the `pre-rewrite` tag. Hunts for lost limits, dropped hedges, merged cases, conditions that did not survive a sentence split. Runs its own token diff over the whole batch.
- **Prose reviewer.** Reads for the register: garden-path sentences, clipped sentences that lost their subject, fragments where the original had a full sentence, repeated openers, "therefore" chains, arrow chains in place of sentences.
- **Integrator.** Applies accepted findings and rejects the rest with a reason.

**Rejection rule.** When a finding contradicts the style guide, the guide wins inside the batch. If the finding is right, the fix is a one-row change to the guide on the tooling branch, never a per-file exception. When a finding needs a tooling change (a tokenizer that fails a legitimate edit, a missing notation mapping), defer it to the tooling branch and say so in the PR.

## Gates on the last commit

Build, unit tests, generated files fresh, `prose:check` (no fail-tier hit in an enabled scope, and zero in the batch's files), end-to-end UI tests if the batch touches UI copy, and `rewrite:check --base pre-rewrite <batch paths>` with zero failures.

## PR description template

1. What the batch covers: the file count and every path.
2. The commit layers and what each did.
3. The fact-check summary: files, units, tokens checked, failures, and each warning explained.
4. Findings fixed, one clause each, naming the file.
5. Findings rejected or deferred, each with its rule or tooling reason.
6. The metrics row for the scope before and after.
7. Gates run and their results.
8. How to review one file: `git diff pre-rewrite -- <path>`.

A calibration PR adds: "Conventions for the remaining batches: ...".

## Final PR checklist

- Regenerate every generated file and the pinned-string appendix.
- Run `rewrite:check --base pre-rewrite` over the whole corpus. Expect a few failures where batches interacted; restore the original wording.
- Re-run `prose:fix` on files a later batch rewrote without the mechanical pass.
- Enable every clean scope.
- A before-and-after metrics table, one row per scope.
- The style guide's "Open items": done, deferred, and remaining hit counts per disabled scope.
