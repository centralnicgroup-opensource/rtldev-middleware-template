# Devcontainer frame example — the compose variant

The team runs two fundamentally different container shapes, and no single
`devcontainer.json` covers both:

| Shape                                        | Where it lives                                       | Repositories                              |
| -------------------------------------------- | ---------------------------------------------------- | ----------------------------------------- |
| one container, built from a `Dockerfile`     | **live** at [`.devcontainer/`](../../.devcontainer/) | php-sdk, mcp-dis, the SDKs, this template |
| one service in a multi-service compose stack | [`compose/`](compose/), an example                   | whmcs-src (web + db + mailpit)            |

The single-container shape is the common case, so the template ships it **live**
rather than as a third copy to be pasted in: a repository created from this
template already has a working environment, and this repository's own environment
is that same frame, which is what keeps it honest.

Only the compose variant is an example, because it is the minority shape and
cannot be the default — it needs a `docker-compose.yml` naming services this
template cannot guess.

## Swapping to compose

The compose frame is an overlay, not a replacement — copying it in overwrites
`devcontainer.json` and `Dockerfile` and adds `docker-compose.yml`, while
`.dockerignore` and `env-info.conf` stay valid as they are:

```sh
cp -r examples/devcontainer/compose/. .devcontainer/
grep -rn '{{' .devcontainer/
```

Then replace the `{{TOKENS}}` the grep finds, declare the host mounts as service
volumes in `docker-compose.yml` (they are not `mounts` entries in a compose
frame), and rebuild.

Use compose when the repository needs a service the dev container does not run
itself — a database, a mail catcher, a second web server. Migrating between the
two shapes later is not hard, but it is a rebuild and a change to how host mounts
are declared, so it is worth getting right up front.

## What is in a frame, and what is not

A frame — live or example — holds only what a devcontainer Feature **cannot** set:

- `name`, `workspaceMount` / `workspaceFolder`, `remoteUser`, `shutdownAction`
- `forwardPorts` / `portsAttributes`
- `initializeCommand` — runs on the **host**, before the container exists, and is
  where the bind sources get created so Docker does not create them itself as
  root-owned directories
- the language runtime this repository is written in (php / go / python / java)
- host mounts (or, for the compose frame, the equivalent service volumes)

Everything else — zsh with the team prompt, commitizen, pnpm, **Node LTS**, the
GitHub CLI, the Claude Code CLI, the `gh` credential helper, persistent shell
history, the shared VS Code extension set _and its settings_, dependency
installation, the RTK binary and the on-attach toolchain banner — comes from the
`devbase` Feature. Do not copy those back into a frame; that is exactly the drift
the Feature exists to stop. In particular a frame needs no `customizations` block
at all unless it is adding a language extension of its own.

The full walkthrough, including how to migrate a repository that already has a
hand-maintained `.devcontainer/`, is in
[the devcontainer-features repository](https://github.com/centralnicgroup-opensource/rtldev-middleware-devcontainer-features).
