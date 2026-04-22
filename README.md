# Dev Environment

A VS Code devcontainer with Node.js, C# (.NET 8 & 10), Go, Python, Dyalog APL, and Claude Code pre-installed.

## What's included

### Languages & runtimes

| Toolchain | Version | Notes |
|-----------|---------|-------|
| Node.js | 20 | Base image; npm globals via `/usr/local/share/npm-global` |
| .NET SDK | 8.0, 10.0 | Installed to `/usr/local/dotnet` |
| Go | 1.24.1 | `GOPATH` set to `~/go`, binaries on `PATH` |
| Python 3 | System (Debian) | Includes `pip`, `venv`, and `pipx` |
| Dyalog APL | 20.0 | Unicode build, arm64 and amd64 |

### Tools

- **Claude Code** — native install, auto-updates
- **gh** — GitHub CLI
- **git-delta** — syntax-highlighted diffs
- **fzf** — fuzzy finder
- **jq** — JSON processor
- **zsh** with Powerlevel10k (default shell)
- **docsearch** — full-text search over Dyalog APL documentation (pre-built database included)
- **bundle-docs** — rebuild the docsearch database from upstream Dyalog docs
- **nano / vim**

### VS Code extensions

- Claude Code (`anthropic.claude-code`)
- ESLint (`dbaeumer.vscode-eslint`)
- Prettier (`esbenp.prettier-vscode`)
- GitLens (`eamodio.gitlens`)
- Go (`golang.go`)
- Python (`ms-python.python`)

## Prerequisites

- [Docker](https://www.docker.com/)
- [VS Code](https://code.visualstudio.com/) with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

## Getting started

1. Clone this repo and open it in VS Code.
2. When prompted, click **Reopen in Container** (or run `Dev Containers: Reopen in Container` from the command palette).
3. Wait for the container to build. First build takes a few minutes; subsequent opens use the cache.

## GitHub CLI authentication

The container forwards `GH_TOKEN` from your host via `remoteEnv`, so `gh` works without running `gh auth login` inside the container.

### Setup

Add this to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):

```sh
export GH_TOKEN=$(gh auth token)
```

Then reload your shell (`source ~/.zshrc`) or open a new terminal before launching the devcontainer.

### Verifying

Inside the container:

```sh
gh auth status
```

You should see your GitHub account authenticated via the token.

### Why `GH_TOKEN` instead of mounting `~/.config/gh`?

- One line in `devcontainer.json` vs. path wrangling across platforms
- No risk of the container writing back to your host config
- Works identically on macOS, Linux, and Windows (WSL)

## Dyalog documentation search

The container includes `docsearch`, a full-text search tool for the Dyalog APL documentation, with a pre-built database at `~/.bundle-docs/dyalog-docs.db`.

### Usage

Search is a two-step process — find relevant documents, then fetch the one you want:

```sh
# Search by keyword
docsearch -s "index generator"
# Output: ROWID TITLE
# 86 Index Generator R←⍳Y
# 502 Search Functions and Hash Tables

# Fetch full document by rowid
docsearch -r 86
```

### Updating the database

To rebuild from the latest upstream Dyalog documentation:

```sh
bundle-docs update
```

This clones the Dyalog documentation repo, parses the mkdocs structure, and regenerates the SQLite database.

## SSH agent forwarding

VS Code forwards your host SSH agent into the container automatically. No extra configuration is needed.

Make sure your host agent is running and has your key loaded:

```sh
# macOS / Linux
ssh-add -l        # should list your key
ssh-add ~/.ssh/id_ed25519   # if not, add it

# Windows — ensure the OpenSSH Authentication Agent service is running
```

Verify inside the container:

```sh
ssh-add -l
```

## Persistent volumes

Two named volumes survive container rebuilds:

| Volume | Mount point | Purpose |
|--------|-------------|---------|
| `claude-code-bashhistory-*` | `/commandhistory` | Shell history |
| `claude-code-config-*` | `/home/node/.claude` | Claude Code config |

## Customisation

### Timezone

Defaults to `America/Los_Angeles`. Override by setting `TZ` on your host:

```sh
export TZ=Europe/London
```

### Build args

Pinned versions can be changed in `devcontainer.json` under `build.args`:

| Arg | Default |
|-----|---------|
| `GIT_DELTA_VERSION` | `0.18.2` |
| `ZSH_IN_DOCKER_VERSION` | `1.2.0` |

Go and Dyalog versions are pinned as `ARG`s in the Dockerfile (`GO_VERSION`, `DYALOG_APL_VERSION`).
