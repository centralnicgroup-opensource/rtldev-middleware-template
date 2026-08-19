# Workflow stubs

Nothing in this directory is active — GitHub only reads workflows from
`.github/workflows/`. Copy what applies and delete the rest.

| File                                                                     | Trigger                | Purpose                                                    |
| ------------------------------------------------------------------------ | ---------------------- | ---------------------------------------------------------- |
| [`lint.yml`](lint.yml)                                                   | pull request           | Linters, analysers, secret scanning, dependency CVE audit  |
| [`test.yml`](test.yml)                                                   | pull request           | Test matrix, gated by a paths filter                       |
| [`release.yml`](release.yml)                                             | push to default branch | semantic-release                                           |
| [`daily-node-dependency-refresh.yml`](daily-node-dependency-refresh.yml) | daily cron             | Full `pnpm update`, **only if** `npm` is out of Dependabot |

## They are thin on purpose

The real jobs live in
[rtldev-middleware-shareable-workflows](https://github.com/centralnicgroup-opensource/rtldev-middleware-shareable-workflows),
so the matrix, caching, coverage upload and CVE gating are maintained once rather than
copied into a dozen repositories and left to drift. If you catch yourself adding real
logic to one of these stubs, it belongs in the shared workflow instead.

Pick the shared workflow that matches this repository's language and replace the
`{{SHARED_*_WORKFLOW}}` tokens — `php-sdk-lint.yml`, `node-sdk-test.yml`,
`go-sdk-release.yml` and so on. The full list is that repository's
`.github/workflows/` directory.

## The `ci-success` job

`lint.yml` and `test.yml` each end with one. It exists so branch protection can
require a single check rather than naming every job inside the shared workflow —
otherwise a job added there silently stops being required here.

In `test.yml` it treats `skipped` as success, because the paths filter deciding no
tests were needed is a pass. Without that, every docs-only PR is blocked.

## Repository settings these assume

- **Squash merging disabled, rebase merging enabled.** semantic-release reads
  individual commit types to pick the version; squashing collapses them into one.
- **Secrets available to the shared workflows** — they are consumed via
  `secrets: inherit`, so anything missing fails inside the shared workflow rather than
  here, which is worth knowing when debugging.
