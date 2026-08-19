# Project Instructions

<!--
SKELETON — fill this in for your repository and delete this comment.

This file is read in full on every task, so its size is a running cost. Keep it to
rules needed on nearly every task, one imperative line each. Rationale, alternatives
and ticket history belong in docs/agents/*.md, linked from here — never summarised
back into this file, because a summary next to its source is a second copy that
drifts.

Delete any section that does not apply. An empty section is worse than a missing one:
it reads as "we have no rules here" rather than "this does not apply".
-->

## Project Overview

{{DESCRIPTION}}

- **Language / runtime:** {{LANGUAGE}}
- **Package name:** {{PACKAGE_NAME}}
- **Default branch:** {{DEFAULT_BRANCH}}

## Architecture

<!--
Facts only, and only the ones that are not derivable from the source tree in a
couple of greps. A file inventory does not belong here; a rule like "brands are
siblings, not parent/child" does, because nothing in the tree states it.
-->

- {{ARCHITECTURE_RULE}}

## Coding Standards

- **Style:** {{STYLE_STANDARD}} (config: `.github/linters/{{LINTER_CONFIG}}`)
- **Static analysis:** {{ANALYSER}} at {{LEVEL}}
- Use typed signatures and declared return types on all new code.
- {{PROJECT_SPECIFIC_STANDARD}}

### Naming

- Classes: PascalCase
- Methods / functions: camelCase
- Constants: UPPER_SNAKE_CASE

## Testing

- **Framework:** {{TEST_FRAMEWORK}}, config `{{TEST_CONFIG_PATH}}`
- **No real API calls in unit tests.** {{HOW_EXTERNAL_CALLS_ARE_FAKED}}
- {{TEST_CONVENTION}}

### Running tests

```sh
{{TEST_COMMAND}}
{{LINT_COMMAND}}
```

## Build, CI & Policies

- **CI** delegates to
  [rtldev-middleware-shareable-workflows](https://github.com/centralnicgroup-opensource/rtldev-middleware-shareable-workflows).
  The matrix, caching, coverage upload and CVE gating live there — do not reimplement
  them in this repository's workflow stubs.
- **Lockfiles are committed deliberately.** Do not remove or git-ignore them.
- **Devcontainer:** the frame is in `.devcontainer/`; the shared behaviour comes from
  the `devbase` devcontainer Feature and is referenced by version, not copied. Never
  fork its scripts into this repository — put repository-specific setup in this
  repository's own `postCreateCommand`.
- **Runtime version policy:** {{VERSION_POLICY}}

## Git Conventions

- **Commit messages:** Conventional Commits with **mandatory scope** —
  `<type>(<scope>): <summary>`. Never append a `Co-Authored-By:` trailer.
- **Commit type selection:** `fix`/`feat` are reserved for source changes — they
  trigger a release. Everything else uses a non-releasing type: `ci` (workflows,
  devcontainer), `build` (build tooling), `chore`, `docs`, `test`, `refactor`.
- **Breaking changes:** add `BREAKING CHANGE: <summary>` to the commit body after a
  blank line, and document the consumer upgrade path in the same change.
- **Branch creation:** `git checkout {{DEFAULT_BRANCH}} && git pull --ff-only` before
  `git checkout -b` — never branch from a stale local default branch or another
  feature branch.
- **Branch naming:** prefix with the Jira issue ID — `RSRMID-1234/short-description`.
- **Pull requests:** include the Jira issue link in the description; after opening,
  add the PR URL as a comment on the Jira issue.
- **Merging:** rebase-merge (`gh pr merge --rebase`). Squash merges are disabled at
  the repository level.

## Important Paths

<!-- Only paths that are NOT guessable from the tree. Delete the rest. -->

| Path                   | Purpose                     |
| ---------------------- | --------------------------- |
| `.github/linters/`     | Linter and analyser configs |
| `{{TEST_CONFIG_PATH}}` | Test runner configuration   |
| `{{NON_OBVIOUS_PATH}}` | {{WHY_IT_IS_NOT_GUESSABLE}} |

## Atlassian / JIRA

Work is tracked in **Jira Cloud**, project `RSRMID`, component `{{JIRA_COMPONENT}}` —
not GitHub Issues.

- **Descriptions must be ADF** (Atlassian Document Format, JSON) — never markdown,
  which renders literal `\n`.
- **Log time before Done:** an issue will not stay in **Done** without a worklog —
  automation stamps `missing-time-spent` and reopens it. Sequence: (1) add worklog;
  (2) remove the label; (3) transition to Done. Ask when the amount is not obvious.

## Tool-Output Hygiene

Every tool result is spent context, so prefer the bounded tool over the shell dump. A
`PreToolUse` hook (`.claude/hooks/tool-output-hygiene.sh`) denies the three worst
shapes and names the replacement — if it fires, take the replacement rather than
working around it.

- **Searching:** the **Grep** tool with `head_limit`, plus
  `output_mode: "files_with_matches"` when only locations matter. An unbounded
  `grep -rn` is never acceptable; where Grep is unavailable, bound the shell form
  (`| head -30`, `-l`, `-c`).
- **Reading part of a file:** **Read** with `offset`/`limit`, never `sed -n '<from>,<to>p'`
  or a bare `cat`.
- **Noisy commands:** bound the output at the source — `git diff --stat` before any
  full `git diff`.
- **MCP calls:** batch field updates into a single call, and never re-fetch an issue
  or page you just mutated — the write response already confirms it.
- `BASH_MAX_OUTPUT_LENGTH` / `MAX_MCP_OUTPUT_TOKENS` in `.claude/settings.json` only
  truncate; a truncated result is a signal you asked the wrong way, not a result to
  work from.

## Model Routing

Opus decides, Sonnet implements: plan and review in the main thread, hand the
mechanical work down. Definitions live in `.claude/agents/`.

- **Implementation** of an already-settled change goes to the `implementer` subagent
  (Sonnet). Trivial one-line edits stay inline — the delegation overhead outweighs
  the saving.
- **Review** goes to the `reviewer` subagent (Opus, pinned so review quality never
  silently drops to a cheaper model), or stays in the main thread. Never route a
  review to Sonnet.
- **Planning and architecture calls** stay on Opus.
- **Fan-out reads** go to `Explore` or `general-purpose`, so the file dumps land in
  the subagent's context instead of this one.
- A subagent reports conclusions, not file contents.

## Do NOT

- Add dependencies without explicit request
- Add `@author` tags to docblocks
- Add `Co-Authored-By:` trailers to commit messages
- Fork the `devbase` devcontainer Feature's scripts into this repository
- {{PROJECT_SPECIFIC_PROHIBITION}}
