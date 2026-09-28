# niri-lay

Save and restore niri workspace layouts as named presets.

I run niri (with DMS) on Arch Linux. Every time I sat down to work I arranged the same few
terminals by hand again — same columns, same widths, same stacking. I got tired of
re-planning the layout every session, so I had an AI write this small script: it saves a
layout I set up once, and puts it back later.

That's the whole idea. It is **not** a session manager — it doesn't snapshot your desktop
or reopen your apps. It only stores and rebuilds the *shape*.

```bash
lay save study     # store the current workspace layout as "study"
lay                # list presets
lay study          # hop to an empty workspace and rebuild "study"
lay add study      # fill the current workspace — the terminal you typed in becomes cell 1
lay switch study code   # if the layout looks like "study" switch to "code", and vice versa
lay edit study     # edit the preset JSON in $EDITOR
lay rm study       # delete a preset
```

Restoring only opens terminals; it runs nothing inside them. You type your own content.

## Requirements

- [niri](https://github.com/niri-wm/niri) — uses `niri msg` IPC and these actions:
  `spawn`, `consume-or-expel-window-left`, `set-column-width`, `focus-column`, `focus-window`
- Python ≥ 3.10 (standard library only, no dependencies)
- a terminal to spawn (default `kitty`; override with `LAY_SPAWN=foot lay study`)
- `$EDITOR` for `lay edit` (default `nvim`)

Tested on Arch Linux with niri 26.04 and kitty. It should work on any niri build exposing
the actions above.

## Install

```bash
git clone <this repo> ~/.config/niri/niri-lay     # lives next to config.kdl, as a niri plugin
ln -s ~/.config/niri/niri-lay/lay ~/.local/bin/lay
```

## How it behaves

- **`save <name>`** — reads every window's column position and tile size in the focused
  workspace; stores the column widths as ratios *and* as absolute pixels, plus per-column
  stacking heights and the `app_id` of each cell.
- **`<name>`** — hops to an empty workspace below (niri creates one), spawns terminals,
  merges them into columns, then sets widths. If the current output width equals the saved
  one, widths are applied in pixels (exact); otherwise it falls back to percentages so it
  still works on another monitor.
- **`add <name>`** — never leaves the current workspace: existing windows fill the first
  column, missing terminals are spawned, and focus is put back on the terminal you ran the
  command in.
- **`switch A B`** — compares the current layout against A and B (column count + widths).
  Looks like A → you get B; looks like B → you get A; looks like neither → it refuses and
  touches nothing.
- **`--apps`** — at restore time, open the recorded `app_id` for each cell instead of a
  terminal (falls back to the terminal when the binary isn't in `PATH`).
- **`--json`** — print `presets.json` as-is, for scripting.

Presets live in `~/.config/niri-lay/presets.json`. One file — copy it to a new machine and
your presets come along.

| field | meaning |
| --- | --- |
| `count`, `width` | windows in the column, width as a ratio of the output |
| `width_px` | absolute width at save time — used when the output width matches |
| `heights` | per-window height ratios inside a stacked column |
| `apps` | program in each cell, used by `--apps` |

## Limitations

- Only terminals are spawned; the script doesn't remember *what* ran inside each cell
  (`--apps` just opens the recorded program list once).
- Floating windows and multi-monitor setups are not handled.
- No tabbed/grouped columns, no window rules.
- Restoring spawns several terminals — it hops to an empty workspace first, so it won't
  shove your current work aside. `add` and `switch` work in place and refuse when the
  layout doesn't fit.
- `switch` needs the current layout to match one of the two presets; there's no fuzzy match.

## Disclaimer

**This project was written by an AI, at my request.** I'm a student; this is the first tool
I've published. The idea, the requirements and the testing are mine, the code is mostly the
AI's — please treat it as a learning project rather than mature software.

- It's a single Python script (~300 lines, stdlib only). Read it before you run it.
- It only talks to niri over IPC. It does not touch your `config.kdl`, and it does not
  delete files.
- Tested only on my setup (Arch Linux, niri 26.04, kitty). Rough edges are likely.
- No warranty of any kind — see [LICENSE](LICENSE). Use it at your own risk.

Bug reports, corrections and ideas are very welcome — issues and PRs are open. If something
is wrong or badly written, saying so is genuinely helpful; I'm here to learn.

Chinese version: [README.zh-CN.md](README.zh-CN.md)

## License

[MIT](LICENSE)
