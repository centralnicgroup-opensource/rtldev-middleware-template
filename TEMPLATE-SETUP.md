# Template setup checklist

You created a repository from `rtldev-middleware-template`. Work through this once,
then **delete this file** — its absence is how anyone can tell the setup finished.

Placeholders are written as `{{TOKEN}}`, and come in two kinds:

- **Identity tokens** — the seven in the table below. The same value everywhere, so
  replace them globally.
- **In-place prompts** — everything else (`{{TRAP_1}}`, `{{TEST_COMMAND}}`,
  `{{SHARED_LINT_WORKFLOW}}`, `{{DB_PASSWORD}}`, …). Each is a question to answer
  where it sits, in `CLAUDE.md`, the agent briefs, the workflow stubs and the compose
  frame. There is no global value for them, which is why they are not tabulated.

Either way, this finds everything still outstanding:

```sh
grep -rnE '\{\{[A-Z_]+\}\}' --exclude-dir=.git --exclude=TEMPLATE-SETUP.md .
```

The pattern is anchored to `UPPER_SNAKE_CASE` on purpose: a bare `{{` also matches
every GitHub Actions `${{ … }}` expression in the workflows, which is not something
to replace.

## 1. Identity

Replace these tokens wherever they appear:

| Token                | Meaning                                   | Example                              |
| -------------------- | ----------------------------------------- | ------------------------------------ |
| `{{REPO_NAME}}`      | Repository name                           | `rtldev-middleware-node-sdk`         |
| `{{PROJECT_NAME}}`   | Human-readable project name               | `Node SDK`                           |
| `{{PROJECT_SLUG}}`   | Short lowercase slug, for container names | `nodesdk`                            |
| `{{DESCRIPTION}}`    | One-line description                      | `Node.js SDK for the Team Internet…` |
| `{{PACKAGE_NAME}}`   | Published package name, if any            | `@team-internet/node-sdk`            |
| `{{LANGUAGE}}`       | Primary language                          | `TypeScript`                         |
| `{{DEFAULT_BRANCH}}` | Default branch                            | `main`                               |

Files that carry them: `package.json`, `README.md`, `CLAUDE.md`,
`.github/SECURITY.md`, and whatever you copy out of `examples/`.

- [ ] Tokens replaced
- [ ] `README.md` rewritten for this project — the template's own README describes
      the template, so it is a starting structure, not content to keep
- [ ] `LICENSE` reviewed (MIT unless there is a reason)

## 2. Devcontainer

`.devcontainer/` is **live and already builds** — the single-container frame every
repository except whmcs-src uses, consuming the shared `devbase` Feature. Nothing
needs copying, and it contains no `{{TOKENS}}`, so you can open the repository in
its container before working through the rest of this list.

What it still wants from you:

- [ ] `"name"` in `devcontainer.json` changed from `dstack-template` to
      `dstack-{{PROJECT_SLUG}}` — the one hardcoded value in the file, because
      `name` does not support variable substitution the way the workspace paths do
- [ ] This repository's language feature uncommented in `devcontainer.json`
      (php / go / python / java — Node needs no entry, `devbase` installs LTS as a
      Feature dependency because prettier, husky and the quality gate run on it in
      every repository)
- [ ] `.devcontainer/Dockerfile` — the commented example layer deleted, or replaced
      with the system packages this repository genuinely needs
- [ ] `.devcontainer/env-info.conf` left as-is (every line is commented and the
      defaults are correct), tuned, or deleted
- [ ] Container rebuilt and verified: prompt renders, `devbase-env-info` reports the
      right versions, `cz` launches the commit prompt, `git push` authenticates
      through `gh` (the helper is written to this repository's `.git/config`, not to
      your global one — `~/.gitconfig` is bind-mounted from the host and must stay
      untouched)
- [ ] `.devcontainer/devcontainer-lock.json` generated with
      `npx @devcontainers/cli upgrade --workspace-folder .` and **committed**. Neither
      `devcontainer build` nor `devcontainer up` writes it, and neither does VS Code's
      "Reopen in Container" — `upgrade` is the only thing that does, so this is an
      explicit step, not a by-product of the first build. The file pins the feature
      digests behind the `:1` tags and is what Dependabot's `devcontainers` ecosystem
      bumps; uncommitted, there is nothing to raise a PR against and the pinning is
      silently lost. `upgrade` only reads Feature metadata from the registry — it needs
      no Docker and builds nothing, so it can be run from inside the devcontainer
      itself, and it is the right command after any manual edit to the feature list

**Only if this repository needs a database, mail catcher or second web server**,
swap in the compose frame — it overlays the live one:

```sh
cp -r examples/devcontainer/compose/. .devcontainer/
grep -rn '{{' .devcontainer/
```

- [ ] Compose frame copied and its tokens replaced
- [ ] Host mounts moved into `docker-compose.yml` as service volumes — a compose
      frame declares them there, not in `mounts`

Options, migration steps and troubleshooting: [the devcontainer-features repository](https://github.com/centralnicgroup-opensource/rtldev-middleware-devcontainer-features).

## 3. Dependabot

`examples/dependabot.yml` is a superset. **Prune it** — a `directory:` that does
not exist raises a per-entry error in the repository's Dependabot tab, so leaving
unused ecosystems in place produces permanent noise rather than being ignored.

```sh
cp examples/dependabot.yml .github/dependabot.yml
```

- [ ] Copied and pruned to the ecosystems this repository actually has
- [ ] `devcontainers` and `docker` ecosystems **kept** — `.devcontainer/` is live in
      every repository created from this template, so both entries apply from day
      one. `devcontainers` is what raises the `devbase` update PRs; `docker` tracks
      the base image in `.devcontainer/Dockerfile`
- [ ] The `npm` decision made consciously — read the comment in the file first

## 4. Workflows

```sh
cp examples/workflows/lint.yml .github/workflows/
cp examples/workflows/test.yml .github/workflows/
cp examples/workflows/release.yml .github/workflows/
```

- [ ] Copied the ones that apply, deleted the rest
- [ ] Left `.github/workflows/quality.yml` in place — it is live, language-agnostic
      (prettier + actionlint + shellcheck) and needs no edits
- [ ] `uses:` lines point at the right shared workflow for this language
- [ ] Branch names match this repository's default branch
- [ ] `daily-node-dependency-refresh.yml` copied if the repository has a
      `pnpm-lock.yaml` and you left `npm` out of Dependabot

## 5. Linters

`.github/linters/` holds configs for several languages.

- [ ] Deleted the ones for languages this repository does not use
- [ ] Remaining configs' paths checked — several reference `../../src` and
      `../../tests` relative to `.github/linters/`
- [ ] Lint scripts wired up in `package.json` / `composer.json`, since the shared
      workflows invoke them through those rather than running the linters directly

## 6. Hygiene files

- [ ] `.editorconfig` — `indent_size` set for this language (4 for PHP, 2 for JS/TS)
- [ ] `.gitignore` — language-specific ignores added, unused blocks removed
- [ ] `.gitattributes` — `export-ignore` lines reviewed if this ships as a package;
      irrelevant otherwise
- [ ] `.prettierignore` — generated and vendored paths listed
- [ ] `package.json` — name, description, maintainers, `engines`, `lint-staged`
- [ ] `codecov.yml` — coverage target agreed, or the file deleted if coverage is not
      reported

## 7. Claude Code

- [ ] `CLAUDE.md` filled in — it is a skeleton with prompts in `{{…}}`, not a
      document to keep as-is
- [ ] `.claude/settings.json` — allowlist extended with this repository's read-only
      commands and safe scripts, nothing that writes
- [ ] `.claude/agents/` — the implementer/reviewer briefs adjusted to this
      repository's language and traps, or deleted if not wanted
- [ ] `.claude/hooks/tool-output-hygiene.sh` — kept or deleted deliberately
- [ ] RTK hook in `.claude/settings.json` — kept (it self-guards, so it is inert without
      the binary), and any personal `rtk hook claude` removed from your own
      `~/.claude/settings.json` so the two do not both rewrite the same call

## 8. GitHub repository settings

"Use this template" copies files, never settings, so a new repository starts on
GitHub's defaults — squash merging on, no protections, no topics. Most of that is
declared in `.github/repo-settings.conf` and reconciled by a script:

```sh
pnpm repo:settings          # report drift, change nothing
pnpm repo:settings:apply    # make GitHub match the file (needs admin)
```

- [ ] `.github/repo-settings.conf` reviewed — description, topics, and the
      `HAS_ISSUES` / `RULESET_ENABLED` decisions
- [ ] `IS_TEMPLATE=false` in that file. It ships as `true` because the file comes from
      a template repository, where that is correct; a repository created from one is
      not itself a template. Applying without changing it puts a "Use this template"
      button on this project
- [ ] `pnpm repo:settings:apply` run, and `pnpm repo:settings` then comes back clean
- [ ] `.github/CODEOWNERS` reviewed
- [ ] Secrets and variables the shared workflows need created. List their names in
      `EXPECTED_SECRETS` / `EXPECTED_VARIABLES` so the drift check reports a missing
      one; values are never in the file

### Branch protection — once per organisation, not per repository

An organisation ruleset covers repositories that do not exist yet, so a new
repository is protected the moment it is created and there is nothing to remember
here. Create it once (needs GitHub Team or Enterprise Cloud):

- Target repositories matching `rtldev-middleware-*`
- Target ref `~DEFAULT_BRANCH`, not a literal branch name — it then follows a
  default-branch rename instead of silently protecting nothing
- Rules: block deletion, block force-push, require a pull request with 1 approval,
  require the `Quality: completed` check, require linear history

- [ ] Organisation ruleset in place — verify with `gh api orgs/OWNER/rulesets` —
      **or**, if org rulesets are not available, `RULESET_ENABLED=true` in the conf
      and applied per repository

## 9. Finish

- [ ] `grep -rnE '\{\{[A-Z_]+\}\}' --exclude-dir=.git .` comes back empty
- [ ] `examples/` deleted, or kept deliberately as a reference
- [ ] This file deleted
- [ ] Initial commit follows the commit convention, e.g.
      `chore(repo): initial setup from rtldev-middleware-template`
