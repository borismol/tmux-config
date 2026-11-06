# Editing ~/.tmux.conf — compact field guide

Lessons from the marker-debugging session (tmux 3.4, Windows Terminal + kitty).

## Verify loop — run after every edit

```bash
tmux source-file ~/.tmux.conf        # or: prefix r
tmux show -g <option>                # what the SERVER stored
# what tmux actually DRAWS (option names use hyphens!):
tmux display -p -t 'session:4' '#{T:window-status-format}' | cat -v
tmux display -p -t 'session:3' '#{T:window-status-current-format}' | cat -v
```

The `T:` probe is the ground truth: an option can look perfect in the file and
in `show -g`, yet still draw garbage. Compare drawn output against what the
screen shows before blaming fonts.

## Rule 1 — no commas inside `#{?...}` branches

tmux splits `#{?flag,true,false}` at **every** top-level comma, including the
comma inside a style:

```tmux
# BROKEN: branch is cut at the comma inside fg=…,bg=…
#{?window_activity_flag, #[fg=colour203]x#[fg=grey,bg=black],}
#   → true branch = "#[fg=colour203]x#[fg=grey"  → leftover "bg=black],"
#     is drawn as literal text on screen

# CORRECT: set colors OUTSIDE, keep only plain text inside
#…#W#[fg=colour203]#{?window_bell_flag, !,}#{?window_activity_flag, ·,}#[restore]
```

This produced the "3:bashbg=colour39]," artifact — identical in every
terminal, so it was never a font issue.

## Rule 2 — flagged tabs get built-in `reverse` on top of your format

`window-status-activity-style` and `window-status-bell-style` default to
`reverse` and apply in addition to your format. Symptom: inverted colors
(dark circle on red background). Fix: neutralize them to the base palette.

```tmux
set -g window-status-activity-style "fg=$C_FG,bg=$C_BG"
set -g window-status-bell-style "fg=$C_FG,bg=$C_BG"
```

## Rule 3 — `#[default]` resets fg AND bg

A space following `#[default]` renders with the terminal default background —
a foreign-colored cell inside your tab ("small dark rectangle"). Restore the
block colors explicitly instead:

```tmux
…marker…#[fg=$C_DIM_FG,bg=$C_DIM] #[fg=$C_DIM,bg=$C_BG]<pill>
```

## Rule 4 — edit glyphs by codepoint, never by copy-paste

Nerd Font / private-use glyphs do not survive typing, copy-paste, or grep
matching. Any string edit touching them fails silently or corrupts them.

```bash
# check what's really in a line:
sed -n '81p' ~/.tmux.conf | od -c
# inventory of symbol codepoints per line:
python3 -c "print(sorted({hex(ord(c)) for c in line if ord(c) > 0x2500}))"
# edit via python with chr(), e.g. chr(0xE0B4), chr(0xF065)
```

Prefer glyphs already proven to render (powerline pills `U+E0B4/E0B6`,
zoom `U+F065`) and plain ASCII/Latin-1 markers (`!`, `·`). Avoid
ambiguous-width characters (`●` rendered double-width in Windows Terminal →
"half circle" bleed into the next cell).

## Rule 5 — a pasted status line lies

Copying the bar mangles glyphs and mixes in surrounding text — never debug
from a paste. Use the `T:` probe (Rule: verify loop) or `od -c` dumps instead.

## Safe-edit checklist

1. Edit → `tmux source-file ~/.tmux.conf` → fix errors if shown inline.
2. `tmux show -g` → confirm stored value.
3. `T:` probe per window state (flagged / current / zoomed) → confirm drawn.
4. Only then trust your eyes on the screen.
