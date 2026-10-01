# Keys cheatsheet

`keys` prints this. `keys <word>` filters it, e.g. `keys split`.
Inside herdr: `ctrl+b i` opens it in a popup, `ctrl+b ?` shows every live binding.

## herdr: how the prefix works

Press `ctrl+b`, let go, then press the key. Written `prefix+x` below.
The mouse also works for everything: click, drag borders, right-click menus.

## herdr: learn these first

| Key                  | Also          | Does                                  |
|----------------------|---------------|---------------------------------------|
| `prefix+?`           |               | show all bindings (`/` to search)     |
| `prefix+c`           | `ctrl+alt+c`  | new tab                               |
| `prefix+v`           | `ctrl+alt+d`  | split right                           |
| `prefix+minus`       | `ctrl+alt+shift+d` | split down                       |
| `prefix+h/j/k/l`     | `ctrl+alt+h/j/k/l` | move to pane left/down/up/right  |
| `prefix+w`           |               | workspace picker                      |
| `prefix+q`           |               | detach (everything keeps running)     |

## herdr: my popups

| Key        | Does                                   |
|------------|----------------------------------------|
| `prefix+d` | critique: live diff of uncommitted changes, else staged, else last commit |
| `prefix+f` | fresh: edit this project               |
| `prefix+a` | lazygit: stage, commit, push, branches |
| `prefix+i` | this cheatsheet                        |

"This project" = the git repo root of the pane you're in, or its folder if
it's not a repo. Same as typing `herdr-here fresh` yourself.

## herdr: panes and tabs

| Key                    | Also           | Does                     |
|------------------------|----------------|--------------------------|
| `prefix+z`             | `ctrl+alt+z`   | zoom pane (toggle)       |
| `prefix+x`             |                | close pane               |
| `prefix+r`             |                | resize mode              |
| `prefix+[`             |                | copy mode (`v` select, `y` copy, `q` quit) |
| `prefix+n` / `prefix+p`| `ctrl+alt+]` / `ctrl+alt+[` | next / previous tab |
| `prefix+1..9`          |                | jump to tab              |
| `prefix+shift+t`       |                | rename tab               |
| `prefix+shift+x`       |                | close tab                |

## herdr: workspaces and session

| Key              | Does                                   |
|------------------|----------------------------------------|
| `prefix+g`       | goto picker: every agent/terminal (`/` search, `b` blocked, `w` working) |
| `prefix+shift+n` | new workspace                          |
| `prefix+shift+w` | rename workspace                       |
| `prefix+shift+d` | close workspace                        |
| `prefix+b`       | toggle sidebar                         |
| `prefix+shift+r` | reload config                          |

## herdr: shell commands

| Command                       | Does                                  |
|-------------------------------|---------------------------------------|
| `herdr`                       | start or reattach                     |
| `herdr server stop`           | actually stop everything              |
| `herdr server reload-config`  | apply config edits (alias `hrdrrc`)   |
| `herdr config check`          | validate config                       |
| `herdr agent list`            | what agents herdr sees (alias `hrdral`) |

## fresh (editor, VS Code-style keys)

| Key                | Does                          |
|--------------------|-------------------------------|
| `ctrl+p`           | command palette / find file   |
| `ctrl+s`           | save                          |
| `ctrl+q`           | quit                          |
| `ctrl+f`           | find in file                  |
| `ctrl+z` / `ctrl+r`| undo / redo                   |
| `ctrl+d`           | add cursor at next match      |
| `ctrl+e`           | focus file explorer           |
| `ctrl+alt+b`       | toggle file explorer (`ctrl+b` belongs to herdr) |

## critique (diff review)

| Command                 | Does                              |
|-------------------------|-----------------------------------|
| `review`                | smart pick: live unstaged diff, else staged, else last commit |
| `critique`              | review uncommitted changes (exits if none) |
| `critique --commit HEAD`| review the last commit            |
| `critique --help`       | everything else                   |

## lazygit (git UI)

| Key            | Does                                   |
|----------------|----------------------------------------|
| `?`            | show all keys for the current panel    |
| `1`..`5` / `[` `]` | switch panel / tab                 |
| `space`        | stage / unstage file                   |
| `c`            | commit                                 |
| `P` / `p`      | push / pull                            |
| `enter`        | open file / commit                     |
| `q`            | quit                                   |

## dotfiles

| Command                         | Does                                    |
|---------------------------------|-----------------------------------------|
| `cd ~/dotfiles && mise bootstrap --dry-run` | preview what would change   |
| `cd ~/dotfiles && mise bootstrap` | apply everything                      |
| `mise bootstrap status --missing` | what's not installed / linked yet     |

Config files in `~/.config/...` are symlinks into `~/dotfiles`, so edits land
in the repo. Commit them from there.
