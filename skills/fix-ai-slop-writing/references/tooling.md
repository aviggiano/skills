# Linter and rewrite tooling

Six scripts and one shared word list, in the repository's own language. All ship in the first PR, before any prose changes. The names below are `package.json` scripts; use the project's task runner if it has another.

## Shared word list

One JSON file (for example `scripts/prose-words.json`) holds `latin`, `notation`, `hardBan` (word → replacement), `reviewWords`, `intensifiersDelete`, `intensifiersReview`, `fillerFrames`, `spellingStems`, `vagueQuantifiers`, `glossary`, `labels`. The linter, the fixer, the metrics script and the fact check all read it, so a new word is a one-place change and the four tools cannot disagree.

## 1. Linter (`prose:check`)

Pipeline per prose unit:

1. **Collect units** per scope: Markdown through a parser (remark, markdown-it, mistune), not regex over raw files; structured fields through the project's loader; UI strings through the language's compiler API (for TypeScript, the TS compiler API over JSX text and named attributes).
2. **Mask** before matching: quoted spans, code blocks and spans, link targets, URLs, math, identifiers, generated lines. Replace each masked span with spaces of equal length so offsets survive. Most linter bugs live here; unit-test every edge case you meet.
3. **Split sentences** on the masked text. The splitter must survive abbreviations, decimals, version numbers, ellipses and quoted sentences.
4. **Run the rules.** Each hit carries rule, scope, location and excerpt.

Output: a table of scope × rule counts, then the first N locations per fail-tier rule. Flags: `--scope`, `--json`, `--all-locations`.

**Two tiers.** Fail-tier rules exit 1 only in scopes listed under `enabled` in a config file. Report-tier rules are counted everywhere and never fail. Every rule is counted in every scope, enabled or not, so the table always shows the whole debt.

**The ratchet.** The config starts with `"enabled": []`:

```json
{
  "enabled": [],
  "sentenceWordCap": { "default": 30, "src/ui": 25 }
}
```

The PR that brings a scope to zero fail-tier hits adds it to `enabled`. From then on CI blocks any regression in that scope, including new AI-written text.

Wire it into CI and into a local gate (`prose:gate` = build + generated-files check + `prose:check`).

## 2. Mechanical fixer (`prose:fix`)

Applies the mechanical tier by script, reusing the linter's masks and the word list. Idempotent. Its output is a separate commit that a human reviews line by line.

## 3. Reflow (`prose:reflow`)

Rewraps Markdown to one sentence per line and changes no words. Test that it leaves tables, lists, code blocks and hard line breaks intact.

## 4. Hash-guarded applier (`rewrite:apply`)

The model never edits prose files directly. It writes **edit files**:

```json
[{ "file": "docs/guides/install.md", "unit": "p:12",
   "before_sha256": "<sha256 of the current text of that unit>", "after": "<new text>" }]
```

- `unit` addresses one prose unit: a paragraph or list-item index in a Markdown file, a heading path, or a field path in structured content.
- `--print-hashes <file>` prints every unit address with its current hash; the rewriter builds edit files from it.
- A directory of edit files is one batch: every hash is checked before any write, and one stale hash refuses the whole batch.
- Writes go through one serializer, so formatting stays stable and the diff shows text only. A round-trip check before the series proves every file is already in canonical form.
- `--base <ref>` re-applies after a rebase. It checks the hash against the ref **and** confirms the current unit still equals the ref's text. Without the second check it overwrites a newer human edit.

## 5. Fact-preservation check (`rewrite:check`)

Compares each file at a git ref with the worktree, or with edit files applied in memory (`--edits <dir>`, so the rewriter checks before it applies). Rules per file or section:

1. Normalize both sides with the word list (Latin, notation, dashes, whitespace).
2. Every number, version, unit, identifier, code span, link target, quoted span, product name and status marker of the before side must appear somewhere in the after side. Facts may move between paragraphs of one section; quotes compare verbatim.
3. A number new to the section fails. The rewriter does not get to invent precision.
4. The count of each condition word (`not`, `no`, `never`, `only`, `unless`, `if`, `must`, `may`, `up to`, `at least`, `more than`) must not drop. A rise is a warning, since a split sentence repeats its condition. An added negation fails ("can be disabled" → "cannot be disabled" inverts a claim with no token lost).
5. Lists keep their length and are compared item by item, or through an explicit alignment map for reorderings. A split or merge of items fails.

`--report <file>` writes JSON with every token checked; the PR description quotes its summary.

State the blind spot in the script header: word-level meaning outside these classes ("up to 10" against "more than 10" can yield the same tokens) is the reviewer's job.

## 6. Metrics (`prose:metrics`)

The step-1 script, regenerated in every batch commit so each PR shows its before and after row.
