# Pitfalls

## Where the quality comes from

- **Calibration does most of the work.** Failure rates drop across the calibration batches and then stay flat through the bulk batches, while the kinds of findings change. So spend human attention on calibration, and do not start the bulk batches until its conventions are guide rules and lint checks.
- **Some failures never go away.** Lost hedges, invented glosses and clipped sentences show up to the last batch. Only the fidelity verifier catches them. Keep it on every batch, sampling end to end, with the checklist in [`batches.md`](batches.md).
- **Findings that stay review-only keep recurring.** Connective stacking and repeated openers come back batch after batch until a measured lint cap exists.

## Tooling defects

- **The linter and the fact check disagree on notation.** A spaced minus is a dash to one and a formula to the other, so no spelling passes both and rewriters fall back to words. Share one tokenizer and test the round trip.
- **The masks misread real text.** Possessives after symbols open phantom quotes; formula masks blank ordinary words; the splitter breaks on abbreviations. Dogfood on the whole corpus before the first batch, and turn each case into a regression test.
- **An auto-delete breaks an idiom.** Deleting "at all" from "if at all" breaks the clause. Every auto-delete word needs negative tests.
- **A stale batch overwrites a newer edit after a rebase.** The applier checks the hash against the base ref only. Also confirm that the current text equals the base text.
- **An added negation passes as a warning.** It inverts a claim without losing a token. Fail on it.
- **The fact check rejects a legitimate rewording around a pinned marker or a range.** Fix the tokenizer through the live tooling loop. Weakening the check for one file hides the next real loss.

## Process defects

- **Tool fixes deferred to the end.** Every batch after the first report hits the same bug, and the fixes pile up in the final PR. Keep the tooling branch live.
- **Generated files committed in batches.** Parallel batches conflict and go stale when merges land out of order. Regenerate once, at the end.
- **A merge from a stale base reverts guide rules.** A batch cut before the calibration conventions landed can squash them away. Merge in manifest order, rebase before each merge, and diff the guide against the tooling tip in the final PR.
- **A batch skips the mechanical pass.** A `--check` mode on the fixer and the reflow tool, run in the final gate, finds it.
- **The glossary drifts.** Old terms return in some batches and not others, because half-rewritten text looks like a standard and prose policies ("expand once per page") read differently to each rewriter. Make the policy a lint rule and tell reviewers that unrewritten files are not a standard.
- **Copied PR descriptions.** A description reused from a previous batch reports that batch's findings. Generate descriptions from the manifest.
- **A rule nobody enforces.** A large share of the corpus can violate it without anyone noticing. Either back it with a check or rewrite it into one a check can enforce, then bulk-fix.
- **Rules copied into other docs.** A skill or template that restates a rule keeps the old version after the guide changes. Link instead.

## What to expect of the results

- Sentence length moves most. A thorough run halves the mean and cuts the share of sentences over 25 words to single digits.
- The corpus gets longer, not shorter, by a few percent. A split sentence repeats its subject and its condition. Word count is not the target.
- Figurative-word counts fall less than the other metrics, because the shared list includes words the domain uses literally. Judge the hard-ban tier, which should reach zero.
- After the series, agents writing to the guide keep new text clean with little help from the gate. The pointer in `AGENTS.md` matters more than the lint.
- Some scopes are not worth finishing in the same series (free-form notes, archived docs). Leave them disabled, list them with counts under "Open items", and let `--changed` mode keep new text in them clean.
