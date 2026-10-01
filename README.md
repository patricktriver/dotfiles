# dotfiles

My macOS setup, applied with [`mise bootstrap`](https://mise.jdx.dev/bootstrap.html).
No nix, no install script: everything is declared in [`mise.toml`](mise.toml).

## New machine

```sh
curl https://mise.run | sh
~/.local/bin/mise bootstrap --from https://github.com/patricktriver/dotfiles.git --from-dir ~/dotfiles
```

## Day to day

```sh
cd ~/dotfiles
mise bootstrap --dry-run   # preview
mise bootstrap             # apply
```

Config files are symlinked from here, so editing `~/.config/herdr/config.toml`
edits `herdr/config.toml` in this repo. Commit from `~/dotfiles`.

Add a brew package: a `"brew:<name>"` or `"brew-cask:<name>"` line under
`[bootstrap.packages]`, then `mise bootstrap`.

## What's here

| Path                | Linked to                                  |
|---------------------|--------------------------------------------|
| `mise/config.toml`  | `~/.config/mise/config.toml` (global tools) |
| `zsh/`              | `~/.zshrc`, `~/.zprofile`, `~/.zshenv`     |
| `git/`              | `~/.gitconfig`, `~/.config/git/ignore`     |
| `herdr/config.toml` | `~/.config/herdr/config.toml`              |
| `fresh/config.json` | `~/.config/fresh/config.json`              |
| `ghostty/`          | Ghostty's config                           |
| `claude/`           | generic skills + commands in `~/.claude` (work ones stay local) |
| `bin/keys`          | `~/.local/bin/keys`                        |
| `bin/herdr-here`    | runs herdr popups in the current project   |
| `bin/review`        | critique, picking a view that has something to show |
| `help.md`           | the cheatsheet `keys` prints               |

## Not in this repo (machine-local, never committed)

- `~/.zshrc.local`: sourced at the end of `.zshrc`
- `~/.gitconfig.local`: git hooks
- `mise.local.toml`: optional extra packages for one machine (gitignored)

A gitleaks pre-commit hook (`.githooks/`) blocks commits that look like they
contain secrets.
