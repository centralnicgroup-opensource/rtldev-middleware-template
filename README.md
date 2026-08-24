# rtldev-middleware-template

[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

The GitHub **template repository** for RTLDEV middleware projects. Creating a
repository from this one gives you the conventions the team already agreed on —
linters, devcontainer, contribution docs, CI wiring, Claude Code setup — instead of
a blank repository and a week of copying files out of a neighbour.

> **New repository?** Work through [TEMPLATE-SETUP.md](TEMPLATE-SETUP.md). It lists
> every file that needs a decision, and gets deleted when you are done.

## What is in here

| Path                                                             | What it gives you                                                                                                                                     |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`.devcontainer/`](.devcontainer/)                               | **Live** single-container devcontainer consuming the shared `devbase` Feature — builds as-is, no tokens.                                              |
| [`examples/devcontainer/`](examples/devcontainer/)               | The multi-service compose frame, for repositories that need a database or mail catcher alongside.                                                     |
| [`examples/workflows/`](examples/workflows/)                     | CI stubs that delegate to the shared reusable workflows.                                                                                              |
| [`examples/dependabot.yml`](examples/dependabot.yml)             | Annotated superset of the ecosystems we use; prune to what the repository has.                                                                        |
| [`.github/linters/`](.github/linters/)                           | Linter and analyser configs, at the paths the shared workflows expect.                                                                                |
| [`.github/workflows/quality.yml`](.github/workflows/quality.yml) | **Live** language-agnostic gate: prettier, actionlint, shellcheck.                                                                                    |
| [`.github/repo-settings.conf`](.github/repo-settings.conf)       | Declared GitHub repository settings — merge buttons, topics, security toggles — reconciled by [`scripts/repo-settings.sh`](scripts/repo-settings.sh). |
| [`.claude/`](.claude/)                                           | Claude Code settings (incl. the RTK hook), the tool-output-hygiene hook, and the implementer/reviewer subagents.                                      |
| [`CLAUDE.md`](CLAUDE.md)                                         | The per-repository agent brief, as a fill-in skeleton.                                                                                                |
| [`CONTRIBUTING.md`](CONTRIBUTING.md)                             | Contribution process and the Code of Conduct.                                                                                                         |
| [`.github/SECURITY.md`](.github/SECURITY.md)                     | Vulnerability reporting policy and response targets.                                                                                                  |

Root-level hygiene files (`.editorconfig`, `.gitattributes`, `.gitignore`,
`.prettierrc`, `.prettierignore`, `.husky/`, `codecov.yml`, `package.json`) are
live, not examples: they apply to a new repository as-is, and only need pruning
where they mention a language the repository does not use. So is
[`.devcontainer/`](.devcontainer/) — see below.

## Nothing under `examples/` is active

One rule, so there is never a question of whether a file is doing something: a
file under `examples/` is inert. GitHub does not read workflows from there,
Dependabot does not read `examples/dependabot.yml`, and VS Code does not read
`examples/devcontainer/`. Copy what you need into the real location — the paths are
in [TEMPLATE-SETUP.md](TEMPLATE-SETUP.md) — and delete the rest.

The reason they are examples rather than live files is that each one needs a
decision the template cannot make: which language ecosystems Dependabot should
watch, which of lint/test/release apply, whether the repository needs a whole
compose stack rather than one container.

Where a sensible default _does_ exist, the template ships it live instead —
`quality.yml` because it is language-agnostic, and `.devcontainer/` because every
repository but whmcs-src is a single container. A default you adjust beats a blank
that has to be filled before anything runs.

## The devcontainer setup

Split in two, because the two halves have opposite lifecycles:

- **The frame** (`.devcontainer/devcontainer.json`, `Dockerfile`, compose file) is
  per-repository and _should_ diverge — it carries the container name, workspace
  paths, forwarded ports and language toolchains. It is **live** in
  [`.devcontainer/`](.devcontainer/): the single-container shape, ready to build,
  with the language features commented out for you to uncomment. Edit it freely.
  Repositories that need a service stack overlay
  [`examples/devcontainer/compose/`](examples/devcontainer/compose/) instead.
- **The behaviour** (zsh and the team prompt, commitizen, pnpm, the `gh` credential
  helper, persistent shell history, dependency installation, the on-attach
  toolchain banner) is the `devbase` **devcontainer Feature** and _must not_
  diverge. It is referenced by version, not copied.

The Feature lives in its own repository,
[rtldev-middleware-devcontainer-features](https://github.com/centralnicgroup-opensource/rtldev-middleware-devcontainer-features),
and that is where its options, migration steps and troubleshooting are documented. The
split follows the same logic as the one above: a template repository is copied once and
then diverges by design, while a published Feature must never diverge — opposite
lifecycles, so they do not belong in one repository.

Both frames already reference it, pinned to the major so Dependabot's
`devcontainers` ecosystem can move the digest in `devcontainer-lock.json`:

```text
ghcr.io/centralnicgroup-opensource/rtldev-middleware-devcontainer-features/devbase:1
```

`devbase` declares **Node LTS, the GitHub CLI and the Claude Code CLI** as Feature
dependencies, so no frame lists any of the three. All are devbase's own plumbing
rather than a repository choice — the `gh` credential helper cannot work without
`gh`, the RTK binary is a Claude Code proxy that does nothing without `claude`, and
Node carries the language-agnostic tooling every repository has whatever its product
language is (prettier, husky, lint-staged, and the quality gate that runs them). None
takes options a repository would want to vary. The consequence is that their digests
move with a `devbase` release rather than with a per-repository Dependabot PR.

A frame must therefore **not** re-declare `node:2` to move the Node version. The
Feature specification treats one Feature id with two different option sets as two
distinct Features, so a frame pinning `node:2` at anything other than devbase's `lts`
installs Node twice into the same nvm path. A repository that genuinely needs a
different major raises it against the Feature — which is the point of it having moved
in. Language runtimes proper (php, go, python, java) stay per-repository and are
declared in the frame; `devbase` lists them in `installsAfter`, so its post-create
runs once they are in place.

Because [`.devcontainer/`](.devcontainer/) is live, this template is itself a
consumer of the Feature — the same coordinate a new repository gets. If a published
`devbase` release breaks, it breaks here too, which is the point.

`devbase` **1.2.2** is the current release and the ghcr package is public, so this
resolves anonymously — no registry login needed to build. Tags available: `1`, `1.0`,
`1.0.0`, `1.1`, `1.1.0`, `1.2`, `1.2.0`, `1.2.1`, `1.2.2`, `latest`. Pin the major
(`:1`) and let the lockfile carry the digest.

## CI

Workflows delegate to
[rtldev-middleware-shareable-workflows](https://github.com/centralnicgroup-opensource/rtldev-middleware-shareable-workflows),
so the matrix, caching, coverage upload and CVE gating live there rather than being
copied into every repository. The stubs in
[`examples/workflows/`](examples/workflows/) are thin on purpose — if you find
yourself adding real logic to one, it probably belongs in the shared workflow.

Repository-level linter configs stay here, at `.github/linters/`, because the
shared workflows invoke them through the consuming repository's own scripts
(`composer phpcs`, `composer phpstan`, …).

One workflow is **live** rather than an example:
[`quality.yml`](.github/workflows/quality.yml) runs prettier, actionlint and shellcheck.
It is language-agnostic, so it is correct in a new repository from day one with nothing to
fill in — and it is named differently from `examples/workflows/lint.yml` precisely so that
copying that one in cannot overwrite it. A repository wants both: the universal checks
here, and its language's analysers from the shared workflow.

## Repository settings

GitHub's template mechanism copies files, never settings — so a new repository inherits
none of the merge-button, topic or security configuration, no matter what this template
says. [`.github/repo-settings.conf`](.github/repo-settings.conf) declares them and
[`scripts/repo-settings.sh`](scripts/repo-settings.sh) reconciles them:

```sh
pnpm repo:settings          # report drift, change nothing
pnpm repo:settings:apply    # make GitHub match the file (needs admin)
```

Check mode never writes and exits non-zero on drift, which is what
[`repo-settings-drift.yml`](.github/workflows/repo-settings-drift.yml) runs weekly.
Applying is deliberately manual: a workflow with admin over its own repository can be
made to apply a merged change to its own settings, protections included.

The conf ships with its identity fields as `{{TOKENS}}`, and check mode reports an
unanswered one as unconfigured rather than failing on it — otherwise the weekly job
could never pass in this repository, whose identity fields stay placeholders by design.
Everything else is still compared, because the merge, feature and security settings are
the same here as in anything created from here. Apply mode refuses outright while a
token remains, so the literal `{{DESCRIPTION}}` never reaches GitHub.

Branch protection is the exception — it belongs in an **organisation ruleset** targeting
`rtldev-middleware-*`, because that also covers repositories that do not exist yet.
See [TEMPLATE-SETUP.md](TEMPLATE-SETUP.md) section 8.

## RTK

`.claude/settings.json` ships a guarded `PreToolUse` hook for
[RTK](https://github.com/rtk-ai/rtk), the token-optimizing CLI proxy for Claude Code:

```json
"command": "command -v rtk >/dev/null 2>&1 && exec rtk hook claude || exit 0"
```

The two halves are centralised separately, on purpose. The **binary** comes from the
`devbase` devcontainer Feature, so it exists wherever the container is built. The **hook**
is this committed file, so it is reviewed and versioned and identical for the whole team
rather than depending on what each developer happened to configure in their personal
`~/.claude`.

The guard is what makes one file safe in all three places it gets read — host, CI, and
container. Only the container is guaranteed to have `rtk`; unguarded, the hook would exit
`127` on every Bash call elsewhere. Guarded, it is a silent no-op until the binary is
present.

If you already have `rtk hook claude` in your own `~/.claude/settings.json`, remove it —
user and project hooks both fire, and this one rewrites the command, so two copies race on
the same tool call.

## Conventions this template assumes

- **Commits:** Conventional Commits with a mandatory scope — `<type>(<scope>): <summary>`.
  `fix`/`feat` are reserved for source changes, because they trigger a release.
- **Releases:** semantic-release, driven by commit types, via the shared release workflow.
- **Package manager:** pnpm, with a committed lockfile.
- **Formatting:** prettier for everything it understands, enforced in CI and by a
  pre-commit hook, not left as a local nag.

## Maintainers

- **Kai Schwarz** — [KaiSchwarz-cnic](https://github.com/kaischwarz-cnic)
- **Asif Nawaz** — [AsifNawaz-cnic](https://github.com/AsifNawaz-cnic)

## License

MIT — see [LICENSE](LICENSE).
