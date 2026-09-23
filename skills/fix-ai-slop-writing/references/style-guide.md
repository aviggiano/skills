# Writing the style guide

The guide is one file at the repository root (`STYLE.md`). The linter, the fixer, the rewrite prompt and the reviewers all read it. Its sections, in order:

## 1. Register, in one paragraph

Name the register and its five properties. State the two tiers. State that no review rule is applied by word-list replacement: a find-and-replace from "leverage" to "use" produces new slop.

## 2. Scope

A table: file type or directory → the prose the lint reads. Typical rows:

| Source | Prose units |
|---|---|
| Markdown (`docs/**/*.md`, `README.md`) | paragraphs, list items, table cells, headings |
| Front matter | `title`, `description`, `summary` |
| Structured content (YAML, JSON) | the named free-text fields |
| UI source (`src/**/*.tsx`) | JSX text, `title`, `aria-label`, `placeholder`, `alt`, string literals of four or more words |
| Docstrings | the summary line and the description, not the parameter tables |

Then the verbatim list: quoted text, code blocks and code spans, generated files, changelogs, legal text, commit messages, anything pinned to a published version.

## 3. Rules

Every rule has four parts:

```
**M1. No em-dash and no spaced hyphen as a dash.** Split into two sentences. A colon is allowed before a list and in the form `Label: statement`.
Before (`docs/guides/install.md:42`): "The CLI is fast — it caches every artifact locally."
After: "The CLI is fast. It caches every artifact locally."
Check: regex for U+2014, or a spaced hyphen or en-dash between two words, outside quotes, code and math.
```

The before is real text from the repository with its `file:line`, as of the day the guide is written. An invented example teaches the rewriter an invented problem.

### Mechanical tier (fails in enabled scopes; a script can fix it)

| Rule | What it forbids | Fix |
|---|---|---|
| M1 | Em-dash, spaced hyphen used as a dash | Split the sentence, or use `Label: statement` |
| M2 | Latin abbreviations in running prose | `i.e.` → "that is", `e.g.` → "for example", `cf.` → "see", `etc.` → name the rest or cut it; keep citation forms such as `§` and `p.` |
| M3 | A list item or caption with inconsistent case or terminal punctuation | One convention, stated in the guide |
| M4 | Mixed notation for one thing (`>=` and `≥`, `->` and `→`, `~` and "about") | One set; convert only in notation position, never in code or quotes |
| M5 | Unknown lead-in labels (`Note:`, `TL;DR:`, `Key takeaway:`) | A fixed label set per document type |
| M6 | Empty intensifiers | Auto-delete `genuinely`, `literally`, `actually`, `truly`, `the very`; flag `simply`, `just`, `merely`, `really`, `incredibly` for review, since some carry "with no other cause" |
| M7 | Mixed spelling conventions | One convention; a stem list with identifiers and quotes excluded |

Add project-specific fail rules as they appear: a raw internal id in reader-facing prose, a cross-reference that does not resolve, a product name spelled two ways.

### Review tier (instructions for the rewriter, checks for the reviewer; the lint only counts)

- **R1. One idea per sentence.** Target 20 words. Hard cap: 25 in UI copy, 30 in long-form prose, citations excluded. The cap is the one review rule with a fail tier. Split at `;` and at a second independent clause. Every split sentence repeats or inherits its condition. Keep the connective ("therefore", "because") when the causal link matters. List the words not to split at when they bind a condition to a noun (often `which` and a temporal `while`).
- **R2. Negation tails, case by case.** Keep ", not Y" and "rather than" when they mark a scope boundary or a hedge. Restructure the aphoristic ones ("It's not a bug, it's a feature").
- **R3. Literal verbs, two tiers.** A hard-ban list of figurative words with no literal meaning in your domain (`unlock`, `leverage`, `supercharge`, `dive into`, `delve`, `unleash`, `empower`, `seamless`, `robust`, `game-changer`, `landscape`, `tapestry`, `journey`, `navigate` for a non-UI sense, `harness`), each with its replacement. A review-only list of words your domain uses literally (for example `deploy`, `fire` for events, `hold` for locks).
- **R4. Name the actor.** A document "states", a function "returns", a user "clicks". No personified documents, features or models ("the docs believe", "this feature wants"). Keep the passive when the actor is unknown or does not matter.
- **R5. One term per concept.** See Glossary.
- **R6. Numbers in place of vague quantifiers, only when the sentence fixes every parameter.** "Several", "a handful", "significantly" become digits only when the number is true for every case the sentence covers. Include a counter-example from your own repository where a number would narrow a general claim.
- **R7. At most one parenthesis per sentence, holding an id or one value.** A parenthetical with a verb becomes a sentence. A list item is one idea; shorten it inside, never split it into two items.
- **R8. No filler frames.** Cut openers and closers that carry no fact: "It's worth noting that", "In today's fast-paced world", "In conclusion", "Let's explore", "Whether you're a beginner or an expert". State the fact.

## 4. Never change

The most important section. The rewrite is only safe when this list is complete:

- Quoted text and everything inside code blocks and code spans.
- Numbers, versions, units, limits and formulas, value and glyph.
- Identifiers: API names, CLI flags, config keys, file paths, URLs and link targets, anchors, CSS classes, test ids.
- Hedges and status framing: "experimental", "deprecated", "not supported", "may", "only if", "unverified". They may move inside a section. They may not be deleted or weakened.
- Domain vocabulary from the primary source (a spec, a standard, the product's own API). Never replace it with a synonym.
- Strings that tests pin. Generate the list with a script that parses the test files (`getByText`, `getByRole({ name })`, snapshot files, `assert.equal`, `expect(...).toBe`), append it to the guide between markers, and add a unit test that fails when the list is stale. A pinned string changes only with its test, in the same commit.

## 5. Glossary

A three-column table: Use | Do not use | Where (which surfaces). Allow one expansion per page for a term readers may know by its old name ("workspace (formerly project)"). The linter reports the unambiguous rows and never fails on them.

## 6. Labels and layout

- A fixed label set per document type, for example `Warning:` and `Note:` only, or admonition blocks only.
- One sentence per line in Markdown source (semantic line breaks). The rendered output does not change, and a diff shows each sentence on its own line, which is what makes a 25-file batch reviewable.
- Numbered steps: one outcome per step, not one keystroke per step.

## 7. Rewrite workflow and open items

Copy the commit layers and gates from [`batches.md`](batches.md) into the guide so a contributor finds them without the skill. The open-items section is written last (step 6).
