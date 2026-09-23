# Linter and rewrite tooling

One shared text library, six scripts and one word list, in the repository's own language. All ship in the first PR, before any prose changes. The names below are `package.json` scripts; use the project's task runner if it has another.

## Shared text library

Build it first and freeze its interface before any script depends on it. It holds the masks, the sentence splitter, the tokenizer and the word-list loader. The linter, the fixer, the reflow tool, the metrics script and the fact check all import it. A private copy in any one tool drifts: parallel agents that build the tools separately will each write their own splitter, and reconciling them later costs more than building one.

The word list is one JSON file (for example `scripts/prose-words.json`) with `latin`, `notation`, `hardBan` (word → replacement), `reviewWords`, `intensifiersDelete`, `intensifiersReview`, `fillerFrames`, `connectives`, `spellingStems`, `vagueQuantifiers`, `glossary`, `labels`, `abbreviations`.

**One tokenizer for lint and fact check.** The linter decides what is a dash and what is a minus; the fact check decides what is a formula token. When they disagree, no spelling passes both, and rewriters work around it with words ("minus"). Add a round-trip test: for every notation construct in the corpus (spaced minus, Unicode minus, `*`, `×`, ranges such as `10-20`, arrows), at least one spelling has zero lint hits and survives the fact check.

## 1. Linter (`prose:check`)

Pipeline per prose unit:

1. **Collect units** per scope: Markdown through a parser (remark, markdown-it, mistune), not regex over raw files; structured fields through the project's loader; UI strings through the language's compiler API.
2. **Run the identifier rules** (raw internal ids, unresolved cross-references) on the unmasked unit, before any mask hides them.
3. **Mask** before matching: quoted spans, code blocks and spans, link targets, URLs, math, identifiers, generated lines. Replace each masked span with spaces of equal length so offsets survive, but keep sentence-final punctuation at the end of a quotation (`"Done." Restart` must still split after the quote).
4. **Split sentences** on the masked text.
5. **Run the rules.** Each hit carries rule, scope, location, and the original (unmasked) sentence.

Output: a table of scope × rule counts, then the first N locations per fail-tier rule. Flags:

- `--scope <prefix>`: accept a prefix (`src` matches `src/ui`), since agents guess scope names.
- `--files <paths>` and `--changed [<ref>]`: lint only these files or only the diff. This is the mode agents run on their own output, and it lets a disabled scope still block new text.
- `--json`, `--all-locations`.

**Two tiers.** Fail-tier rules exit 1 only in scopes listed under `enabled` in a config file, and in any scope in `--files` or `--changed` mode. Report-tier rules are counted everywhere and never fail. Every rule is counted in every scope, so the table always shows the whole debt.

**The ratchet.** The config starts with `"enabled": []`:

```json
{
  "enabled": [],
  "sentenceWordCap": { "default": 30, "src/ui": 25 }
}
```

The PR that brings a scope to zero fail-tier hits adds it to `enabled`. A PR that creates a new prose surface enables its scope in the same PR.

### Masking and tokenizing: where the bugs are

Dogfooding on the real corpus finds these; each becomes a regression test taken from the corpus:

- A formula mask that treats any word as an operand blanks prose. An operand is a number, a single letter, a symbol, or a token with a digit, underscore or `%`. A formula never starts inside a word or after a hyphen.
- A hyphen or slash chain swallows an id (`user-id/v2`). Mask identifiers before formulas.
- A possessive or prime after `++`, `}`, `)` or a letter (`C++'s`, `f'`) opens a phantom single-quoted span. When you change the quote pattern, count the spans across the whole corpus before and after, and read every span that changed.
- The splitter breaks on abbreviations (`Fig.`, `vs.`, `approx.`), decimals, versions (`1.5x`, `v2.0`) and ellipses, and it misses a sentence that ends at a closing quote or bracket. Keep one abbreviation list for the splitter and the metrics.
- Label rules miss case variants at a sentence start. Match case-insensitively.

## 2. Mechanical fixer (`prose:fix`)

Applies the mechanical tier by script through the shared masks: it edits unmasked text only, it is idempotent, and it has negative tests for every idiom that contains an auto-delete word. Rules that need judgment are never auto-applied. `--check` mode exits 1 when a file would change, so the final gate can prove no file skipped the pass.

## 3. Reflow (`prose:reflow`)

Rewraps Markdown to one sentence per line and asserts that the change is whitespace only. Test that it leaves tables, lists, code blocks and hard line breaks intact.

## 4. Hash-guarded applier (`rewrite:apply`)

The model never edits prose files directly. It writes **edit files**:

```json
[{ "file": "docs/guides/install.md", "unit": "p:12",
   "before_sha256": "<sha256 of the current text of that unit>", "after": "<new text>" }]
```

- `unit` addresses one prose unit: a paragraph or list-item index in a Markdown file, a heading path, or a field path in structured content.
- `--print-hashes <file>` prints every unit address with its current hash.
- A directory of edit files is one batch: every hash is checked before any write, and one stale hash refuses the whole batch.
- It refuses a list edit that duplicates an item, and treats a whitespace-only value as empty.
- Writes go through one serializer, so formatting stays stable and the diff shows text only. A round-trip check before the series proves every file is already in canonical form.
- `--base <ref>` re-applies after a rebase. It checks the hash against the ref **and** confirms the current unit still equals the ref's text. Without the second check it overwrites a newer human edit.

## 5. Fact-preservation check (`rewrite:check`)

Compares each file at a git ref with the worktree, or with edit files applied in memory (`--edits <dir>`, so the rewriter checks before anything is written). It works on a multiset of tokens per file or section:

1. Normalize both sides through the shared library (Latin, notation, dashes, whitespace).
2. Every number, version, unit, identifier, code span, link target, quoted span, product name and status marker of the before side must appear in the after side. Facts may move between paragraphs of one section. A move to another section or file needs an explicit alignment map (`<source unit> → <destination unit>`), and the check compares the pair. Quotes compare verbatim. An ordered pair (`2s → 5s`) is one token, so swapping its sides fails.
3. Numbers compare as a multiset: a number new to the section fails, and so does an extra occurrence of a number already there ("timeout is 10 seconds" gaining "retry 10 times"). The rewriter does not get to invent precision. When a split sentence legitimately repeats a value, the reviewer records it in an allowlist in the check report.
4. First normalize every negation form to one token: `cannot`, `can't`, every `n't` contraction, `none`, `neither`, `nor`, `without`, `no longer`. Then the count of each condition, negation and bound word (`not`, `no`, `never`, `only`, `unless`, `if`, `must`, `may`, `up to`, `at least`, `more than`, `below`) must not drop. A rise is a warning, since a split sentence repeats its condition. An added negation fails ("can be disabled" → "cannot be disabled" inverts a claim with no token lost).
5. Lists keep their length and are compared item by item, or through an explicit alignment map for reorderings.

`--report <file>` writes JSON with every token checked; the PR description quotes its summary.

State the blind spot in the script header: added glosses, actors and claims, and word-level meaning ("up to 10" against "more than 10"), are the reviewer's job. Token counts do not catch them.

## 6. Metrics (`prose:metrics`)

The step-1 script. It runs in CI and in the final PR, not in every batch commit; see the generated-files rule in [`batches.md`](batches.md).
