# Keeping it clean

After the series, new prose comes mostly from agents. They stay in the register when the guide is easy to find and cheap to check, not because CI blocks them. Build for that.

## Point at the guide; never copy it

- **`AGENTS.md` at the root** (and a `CLAUDE.md` that imports it). Agents look for it first. When it is missing, they find the guide only when a prompt happens to name it. One line is enough:

  ```
  All prose follows STYLE.md. Run `npm run prose:check -- --changed` before committing; the lint is the authority.
  ```

- **Every skill, template or prompt that writes prose links to the guide and names the lint command.** It does not restate a rule. A restated rule goes stale the first time the guide changes. Add a CI check that fails when a skill file contains the text of a guide rule.
- **One glossary.** If the project grows a structured glossary, the guide's term table points to it.

## Cover every surface

- Keep a Markdown scope on by default (README, `docs/`, skill files, templates). Agents write these most, and they are the first place slop comes back.
- A PR that creates a new prose surface enables its lint scope in the same PR.
- A disabled scope still blocks new text: CI runs `prose:check --changed` against the base branch, which fails on fail-tier hits in changed units of any scope.

## Check early

- An agent stop hook or a pre-commit hook runs `prose:check --changed`. CI alone reports after the push.
- Keep the lint fast enough that an agent runs it without being asked: seconds, not minutes, on a diff.

## Change a rule, not the text

- When a rule turns out to be wrong or unenforceable, rewrite the rule to what a tool can check, then bulk-fix the corpus through the same batch workflow and fact check. A rule the corpus ignores is worse than no rule.
- Record a changed rule's history in the PR, not in the guide. The guide states the current rule only.
