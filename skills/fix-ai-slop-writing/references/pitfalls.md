# Pitfalls

## Defects this process tends to let through

- **A stale batch overwrites a newer edit after a rebase.** The applier checks the hash against the base ref only. Also confirm that the current text equals the base text.
- **An added negation passes as a warning.** A negation inverts a claim without losing a token. Fail on it, or require a reviewer to clear each warning by name.
- **The masks misread notation.** A spaced minus in `a - b` reads as a dash; `>=` inside a code span gets converted. A spaced hyphen between two operands is notation; before a word it is a dash. Add a unit test for each case you meet.
- **A notation rule misses operator positions.** Count every operator form the corpus uses, and fix the linter and the fixer together.
- **A later batch skips the mechanical pass.** The final PR re-runs the fixer over everything.
- **A structural move gets paraphrased.** Moving text to another field or section is not a rewrite; keep its wording so the fact check can match it.
- **The tokenizer fights a legitimate edit.** Normalizing `v2` to `v2.0` or `-` to `−` fails as a lost or new token. Defer the normalization to the tooling branch rather than weaken the check for one file.
- **Generated files go stale** when squash merges land in a different order than the batches were cut.

## Conventions calibration usually produces

- Keep the original's opener form (fragment or full sentence). Converting one into the other tends to make openers worse.
- No colon after a transitive verb before a list ("It supports: A, B" becomes "It supports A and B" or "It supports these formats:").
- Split an over-long list item into sentences inside the item, never into two items.
- A terminal period on a list item only when the item holds more than one sentence.
- One glossary expansion per page. Unrewritten pages are not a standard to match.
- "Therefore" at most once per paragraph or numbered step.

## What to expect of the results

- Sentence length is the metric that moves most. A thorough run halves the mean and cuts the share of sentences over 25 words to single digits.
- The corpus gets longer, not shorter, by a few percent. A split sentence repeats its subject and its condition. Word count is not the target.
- Figurative-word counts fall less than the other metrics, because the shared list includes words the domain uses literally. Judge the hard-ban tier, which should reach zero.
- Some scopes are not worth finishing in the same series (free-form notes, archived docs). Leave them disabled and list them with counts under "Open items". A disabled scope with a written reason beats a half-rewritten scope that CI claims is clean.
