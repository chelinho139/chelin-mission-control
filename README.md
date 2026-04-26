# chelin-mission-control

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![Platform: macOS](https://img.shields.io/badge/platform-macOS-lightgrey.svg)](#requirements)

> A NetHack-style terminal UI that shows every Claude Code session running on your Mac as a card with an animated mascot, status, % of context used, and the last prompt — and lets you jump straight into the iTerm tab that owns it.

```
                    *** CLAUDE MISSION CONTROL ***
              sessions: 4    refresh: 0.4s ago    14:22:09

+--[ live sessions ]----------------------------------------------------+
|                                                                      |
| +--[ ◆ inventory-api:main ]---+   +--[ ! todo-app:dark-mode ]---+    |
| |Akemi                        |   |Diya                         |    |
| |        ▄█████▄              |   |        ▄█████▄  !           |    |
| |        █●░░░●█  ⠋           |   |        █◉░░░◉█              |    |
| |        ▀█████▀              |   |        ▀█████▀              |    |
| |        WORKING              |   |        WAITING              |    |
| |                             |   |                             |    |
| |     tool: Edit              |   |    needs input              |    |
| |   12.3k tok · 25% ctx       |   |   8.1k tok · 18% ctx        |    |
| |> add a hover state to the …|   |> review the migration plan  |    |
| +-----------------------------+   +-----------------------------+    |
|                                                                      |
| +--[ ✗ marketing-site:main ]--+   +--[ Z pdf-thumb:master ]----+     |
| |Bashir                       |   |Valentina        z           |    |
| |        ▄█████▄  !           |   |        ▄█████▄ Z            |    |
| |        █◉░░░◉█              |   |        █─░░░─█              |    |
| |        ▀█████▀              |   |        ▀█████▀              |    |
| |         STUCK               |   |        ZOMBIE               |    |
| |                             |   |                             |    |
| |  awaiting permission        |   |    no writes 35d            |    |
| |    8.2k tok · 18% ctx       |   |     0 tok · 0% ctx          |    |
| |> deploy the new pricing pa…|   |> rebuild the worker pool    |    |
| +-----------------------------+   +-----------------------------+    |
+----------------------------------------------------------------------+
+--[ detail :: 3c24eeee ]----------------------------------------------+
|       cwd : /Users/you/code/inventory-api                             |
|   session : 3c24eeee-ac77-49b8-a31d-f0349acbc48b                      |
|    branch : main          idle : 2s          model : sonnet-4-6       |
| last user : add a hover state to the cards                            |
| last asst : I added a hover state with a subtle scale transform...    |
+----------------------------------------------------------------------+
 [h/j/k/l]move [f]ocus [enter]detail [o]pen [c]opy [K]ill [r]efresh [q]uit
```

## Features

- **One screen for every Claude Code session** running on your machine, sorted by attention required (waiting → stuck → working → idle → zombie → ended).
- **Animated mascot per card** with a state-driven personality: a braille-spinner working face, a blinking notification face, sleeping Z's for zombies, idle dots — colored to match the status.
- **Stable mascot names** — each session gets a deterministic name from a curated pool of 318 multicultural first names (hashed from `session_id`), so "George" is always that one session and you can talk about cards by name.
- **Status-colored borders** — a STUCK card wears a red border, a WAITING card wears yellow, etc., so attention-needing sessions pop out across the grid before you even read the labels.
- **Accurate state detection** via Claude Code lifecycle hooks — knows when a session is awaiting a permission prompt vs. actually working.
- **% of context shown per card** so you know which sessions are about to bump the context limit.
- **`f`ocus** — jump to the iTerm2 or Terminal.app tab running any session.
- **Zombie cleanup** — surface `claude` processes that have been idle for hours and `K`ill them with one keystroke.
- **Two layouts:** the default card grid, and a classic single-row table via `--rows`.
- **Single-file, stdlib only.** No dependencies, no virtualenv. ~1.4k lines of Python 3.

## Why

If you run multiple Claude Code sessions in parallel terminal tabs, you forget which one is on which branch, which one is waiting for your input, and which one crashed weeks ago and is still hanging. This tool gives you one screen that answers all of those questions and lets you `f`ocus the right tab.

## Installation

```bash
curl -o /usr/local/bin/chelin-mission-control \
  https://raw.githubusercontent.com/chelinho139/chelin-mission-control/main/chelin-mission-control
chmod +x /usr/local/bin/chelin-mission-control
```

Or just download the script and run it: `python3 chelin-mission-control`.

## Usage

```bash
chelin-mission-control            # launch the TUI (cards layout)
chelin-mission-control --rows     # launch with the classic table layout
chelin-mission-control --setup    # install Claude Code hooks (asked on first launch)
chelin-mission-control --teardown # remove the hooks
```

On first launch, the TUI offers to install lifecycle hooks in `~/.claude/settings.json` for accurate status detection. Pick `Y` (recommended), `n` to skip for this run, or `d` to never ask again. You can always run `--setup` / `--teardown` manually later.

Without the hooks, the TUI still works but uses transcript heuristics, which are less accurate — particularly for "awaiting permission" detection.

## Card anatomy

```
+--[ ◆ inventory-api:main ]----+
|Akemi                         |   ← mascot name (stable per session)
|        ▄█████▄               |   ← mascot top
|        █●░░░●█  ⠋            |   ← mascot eyes + animation accessory
|        ▀█████▀               |   ← mascot bottom
|        WORKING               |   ← status verb (centered under face)
|                              |
|      tool: Edit              |   ← status detail
|   12.3k tok · 25% ctx        |   ← cumulative tokens · % of context window
|> add a hover state to the c…|   ← last user prompt (truncated)
+------------------------------+
```

The card border carries the status color. A green border + bold title means the card is currently selected (cursor focus).

The grid auto-flows based on terminal width — 2 cards across at 80 columns, 3 at 120, 4 at 160. If more sessions exist than fit the screen, a `page N/M` footer appears at the bottom of the grid box.

## Mascot animations

| status   | animation                                                       | speed |
|----------|-----------------------------------------------------------------|-------|
| working  | 8-frame braille spinner (`⠋⠙⠹⠸⠼⠴⠦⠧`) with eyes blinking once    | 5 fps |
| waiting  | 2-frame `!` flash with `◉↔●` eyes                                | 2 fps |
| stuck    | same as waiting but faster — urgent flash                       | 3 fps |
| idle     | 3-frame waiting dots `.` → `. .` → `. . .`                       | 1.5 fps |
| zombie   | sleeping Z's that float up and grow                             | 1 fps |
| ended    | sleeping Z's, slower (dormant)                                  | 0.5 fps |

All cards animate against a single shared clock, so they "breathe" in sync.

## Requirements

- macOS (uses `lsof`, `pbcopy`, and AppleScript for tab focusing)
- Python 3.9+
- [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) installed and writing transcripts to `~/.claude/projects/`
- iTerm2 *or* Terminal.app for the `[f]ocus tab` feature (everything else works without it)
- A 256-color terminal for the muted dim/cursor palette (falls back gracefully on 16-color terminals)

Linux support is plausible but untested — only the focus/clipboard/finder integrations are macOS-specific.

## Keybindings

| key                  | action                                                             |
|----------------------|--------------------------------------------------------------------|
| `j` / `k` / arrows   | move cursor down / up (cards: by row of cards; rows: by row)        |
| `h` / `l` / arrows   | move cursor left / right within a row of cards (cards mode only)    |
| `g` / `G`            | jump to top / bottom                                               |
| `enter`              | show transcript tail in a modal                                    |
| `f`                  | focus the iTerm/Terminal tab running this session                  |
| `o`                  | open the project folder in Finder                                  |
| `c`                  | copy the session ID to clipboard                                   |
| `K`                  | send SIGTERM to the session's process (cleans up zombies)          |
| `r`                  | force refresh                                                      |
| `q` / `esc`          | quit                                                               |

## Status legend

| glyph | status      | meaning                                                                        |
|-------|-------------|--------------------------------------------------------------------------------|
| `◆`   | **working** | actively processing a tool call or response                                    |
| `!`   | **waiting** | last entry is an assistant message — needs your input                          |
| `✗`   | **stuck**   | pending tool call with no result for 15s+ — likely awaiting a permission prompt |
| `Z`   | **zombie**  | `claude` process running but no transcript activity for 1h+                    |
| `·`   | **idle**    | quiet session                                                                  |
| `†`   | **ended**   | recent transcript but no live process                                          |

## How it works

- **Session discovery.** Scans live `claude` processes via `ps` and finds each one's working directory via `lsof`. The cwd determines which `~/.claude/projects/<encoded-cwd>/` directory holds the transcript; the most recent `.jsonl` file there is the active session.
- **Status detection (with `--setup`).** Reads the per-session event log at `~/.claude/mission-control/<session_id>.state.jsonl`, populated by Claude Code hooks. Each event (`PreToolUse`, `PermissionRequest`, `Stop`, ...) maps to a status — no parsing or heuristics needed.
- **Status detection (fallback).** Parses the tail of each JSONL transcript looking at the last entry type and pending `tool_use` / `tool_result` pairs. Less accurate than the hook-based path, particularly for permission prompts.
- **% of context.** From the most recent assistant message in the transcript, takes `input_tokens + cache_read_input_tokens` and divides by the model's context window — 200 K by default, 1 M when the model id contains `1m` (e.g. `claude-opus-4-7-1m`).
- **Mascot names.** A 318-name pool is indexed by `md5(session_id) mod len(pool)`. Same session always lands on the same name across reruns; no state file.
- **Tab focusing.** Gets the TTY of the `claude` PID via `ps -o tty=`, then runs an AppleScript that walks every iTerm2 (or Terminal.app) tab looking for a matching `tty` and activates it.

## Privacy

The script reads transcript files containing the full text of your past Claude Code conversations. **Everything stays local** — there is no network code in this project. The card and transcript modal will display real conversation snippets (the last user prompt is shown directly on each card), so use a fresh test session if you make a screencast.

When `--setup` is enabled, the hook handler writes one JSON line per Claude Code lifecycle event to `~/.claude/mission-control/<session_id>.state.jsonl`. The recorded fields are: timestamp, event name, tool name, tool-use id, permission mode, and the notification matcher — **no message content, prompt text, or tool input/output**.

## Limitations

- The `[f]ocus tab` feature only works for sessions running directly in an iTerm/Terminal tab. Sessions inside `tmux`/`screen` will focus the right outer tab but you'll still need to switch panes manually.
- Without `--setup`, "awaiting permission" detection is a heuristic and frequently misclassifies sessions.
- Sessions that started before `--setup` ran will use the heuristic until they're restarted (no historical hook events to read).
- The cwd → project-dir encoding (slashes become dashes) is ambiguous when paths contain real `-` characters; the script disambiguates by probing the filesystem.
- Mascot name pool is 318 entries — duplicate names start appearing around 21 simultaneous sessions (birthday paradox). Fine for normal use, less so if you somehow run 30+ sessions at once.

## Contributing

Issues and pull requests welcome at [github.com/chelinho139/chelin-mission-control](https://github.com/chelinho139/chelin-mission-control). Please include your macOS and Python versions, and a brief description of what you saw vs. expected.

## License

[MIT](LICENSE) © Chelinho
