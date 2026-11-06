# tmux Cheatsheet

**Prefix = `Ctrl+b`** (press, release, then the key). The **PREFIX** badge in the status bar turns red while it's active.

## Sessions (from shell)

| Command | Action |
|---|---|
| `tmux new -s name` | New named session |
| `tmux ls` | List sessions |
| `tmux attach -t name` | Attach to session |
| `tmux kill-session -t name` | Kill it |

| Keys | Action |
|---|---|
| `prefix d` | Detach (session keeps running) |
| `prefix s` | Session chooser |
| `prefix $` | Rename session |
| `prefix ( / )` | Previous / next session |

## Windows

| Keys | Action |
|---|---|
| `prefix c` | New window — **opens in current dir** *(custom)* |
| `prefix 1–9` | Jump to window (numbers **start at 1**) |
| `prefix Tab` | Last used window *(custom)* |
| `prefix n / p` | Next / previous |
| `prefix ,` | Rename window — names **stick**, auto-rename is off |
| `prefix &` | Kill window |
| `prefix w` | Window chooser |

Windows renumber automatically after you close one.

## Panes

| Keys | Action |
|---|---|
| `prefix "` | Split top/bottom — in current dir |
| `prefix %` | Split left/right — in current dir |
| `prefix h j k l` | Move between panes (vim-style; arrows also work) |
| `prefix \` | Last used pane *(custom)* |
| `prefix q` | Show pane numbers (2 s) — type a digit to jump there |
| `prefix z` | Toggle zoom — **ZOOM** badge appears in status bar |
| `prefix Space` | Cycle layouts |
| `prefix { / }` | Swap pane left / right |
| `prefix !` | Break pane out into its own window |
| `prefix x` | Kill pane (confirm) |
| `prefix X` | **Kill all *other* panes** (confirm) *(custom)* |

**Resizing** *(all repeatable — just keep pressing)*:

- `prefix H J K L` — resize by 5 *(custom)*
- `prefix Ctrl+Arrow` — resize by 1, `prefix Alt+Arrow` — by 5

## Scratch popup

`prefix t` → 80%×80% popup terminal attached to a `scratch` session, opened in the current pane's dir. Detach inside it (`prefix d`) or exit the shell and the popup closes. Great for one-off commands — keeps your real windows clean.

## Copy mode & clipboard (vi keys)

| Keys | Action |
|---|---|
| `prefix [` | Enter copy mode (or just scroll with the mouse wheel) |
| `v` | Start selection |
| `y` | Copy → **system clipboard** (OSC 52, works even over SSH) and exit |
| `/` then `n / N` | Search down / next match (`?` for up) |
| `q` | Exit copy mode |

**Mouse:**

- Drag = select + copy on release; the selection stays visible
- When the app grabs the mouse (vim, opencode…): hold **Alt or Shift** while dragging to force tmux selection
- Click switches panes; wheel scrolls history

## Status bar legend

| Element | Meaning |
|---|---|
| Left blue block | Session name |
| `PREFIX` red block | Prefix pressed (red = active) |
| `ZOOM` | Current window is zoomed |
| `!` on a tab | Terminal bell rang in that window (e.g. job finished) |
| `·` on a window | Activity in a background window (silent — just glance) |
| Right side | Current dir · git branch · hostname · clock (updates every 5 s) |

## Maintenance

| Keys / Command | Action |
|---|---|
| `prefix r` | Reload `~/.tmux.conf` *(custom)* |
| `tmux show -g` | Inspect live options |

---

# Best practices for this config

1. **One named session per project** — `tmux new -s work`, switch with `prefix s`. Never attach two clients to the same session casually.
2. **Detach, don't kill** — `prefix d` leaves everything running; killing a session just jumps you to another one instead of dropping out (`detach-on-destroy off`).
3. **Name your windows** (`prefix ,`) — auto-rename is off *by design*, so your names won't be overwritten.
4. **Count from 1** — both window and pane indexes start at 1; don't hunt for a "window 0".
5. **Use `prefix z` + the ZOOM badge** — zoom to focus, and the badge reminds you to unzoom. For quick commands prefer `prefix t` (scratch popup) over temporary splits.
6. **`prefix X` is powerful** — it wipes every pane but the active one. The confirm prompt is your only safety net.
7. **Copying over SSH "just works"** — `y` in copy mode pushes to your local clipboard via OSC 52; kitty/ghostty support this. No xclip plumbing needed.
8. **Fast Esc, real focus events** — 10 ms escape-time and `focus-events on` mean vim's Esc is instant and autoread works; don't add plugins for that.
9. **Splits inherit your directory** — new windows/panes open where you already are, so no `cd` rituals.
10. **Watch for `·`** — background-window activity is silent (no popup spam); check the status bar, or jump with `prefix w`.
11. **After editing the config**: `prefix r`; errors show inline in the message bar. If something breaks badly, `tmux kill-server` and reattach — sessions are lost, so detach your important work first.
12. **50,000 lines of history per pane** — search it in copy mode with `/` instead of re-running things.
