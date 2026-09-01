# Linter configurations

**Delete the configs for languages this repository does not use.** An orphan
`sun_checks.xml` in a Node repository is a config nobody maintains and everybody
eventually trips over.

| File                               | Tool                 | Language   |
| ---------------------------------- | -------------------- | ---------- |
| [`phpcs.xml`](phpcs.xml)           | PHP_CodeSniffer      | PHP        |
| [`phpstan.neon`](phpstan.neon)     | PHPStan (level 9)    | PHP        |
| [`psalm.xml`](psalm.xml)           | Psalm (errorLevel 1) | PHP        |
| [`tsconfig.json`](tsconfig.json)   | TypeScript compiler  | TypeScript |
| [`sun_checks.xml`](sun_checks.xml) | Checkstyle           | Java       |

## Why the paths climb two levels

The PHP configs reference `../../src` and `../../tests` because they are resolved
relative to the config file, not the working directory. Keep them that way: the shared
CI workflows invoke the analysers through the repository's own scripts
(`composer phpcs`, `composer phpstan`, `composer psalm`), which run from the repository
root with an explicit `-c .github/linters/…`. A config that assumes the root instead
resolves to `.github/linters/src`, and analyses nothing.

It will not pass quietly, though. Handed a path that matches nothing — missing, or
present but empty — each analyser exits non-zero and says why:

| Tool    | Exit | Message                      |
| ------- | ---- | ---------------------------- |
| PHPStan | 1    | `No files found to analyse.` |
| Psalm   | 2    | `No files analyzed`          |
| phpcs   | 16   | `No files were checked.`     |

Note that scope is declared in **two** places depending on the tool: `phpstan.neon`
and `psalm.xml` carry their own paths, while `phpcs.xml` declares no `<file>` elements
and takes them on the command line (`… -q src tests`), cwd-relative. Changing what
gets analysed means editing the config for the analysers _and_ the script for the
sniffer.

The one place an empty input really is silent is `xargs --no-run-if-empty` in the
`shellcheck` job of `.github/workflows/quality.yml`, which exits 0 saying nothing —
deliberately, so a repository with no shell scripts passes.

## Wiring them up

The shared workflows call your scripts rather than the linters directly, so a config
here does nothing until the script exists. For PHP, in `composer.json`:

```json
"scripts": {
  "phpcs": "phpcs --standard=.github/linters/phpcs.xml -q src tests",
  "phpstan": "phpstan analyse -c .github/linters/phpstan.neon --no-progress",
  "psalm": "psalm -c .github/linters/psalm.xml",
  "lint": ["@prettier", "@phpcs", "@phpstan", "@psalm"]
}
```

Prettier is intentionally part of `lint` and gated in CI, not left as a local nag —
unformatted Markdown should be a red build.

## Suppressions

Every suppression needs a comment saying what it is for. An uncommented suppression is
indistinguishable from a silenced real bug a year later, and both `psalm.xml`'s
`findUnusedBaselineEntry` and its `findUnusedCode` exist to force that question to be
answered on evidence: a suppression that stops firing becomes an error rather than
lingering "just in case".
