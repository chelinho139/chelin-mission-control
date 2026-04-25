# chelin-mission-control

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![Platform: macOS](https://img.shields.io/badge/platform-macOS-lightgrey.svg)](#requirements)

> A NetHack-style terminal UI that shows every Claude Code session running on your Mac — project, branch, status, idle time, tokens — and lets you jump straight into the iTerm tab that owns it.

```
                    *** CLAUDE MISSION CONTROL ***
              sessions: 5    refresh: 0.4s ago    14:22:09

+--[ live sessions ]----------------------------------------------------+
| ## st  project              branch           status            idle  |
| a  !   todo-app             feat/dark-mode   WAITING needs in  1m23s |
| b  ◆   inventory-api        main             WORKING tool: Edit   2s |
| c  ✗   marketing-site       main             STUCK awaiting per 18s  |
| d  Z   pdf-thumbnailer      master           ZOMBIE no writes 35d    |
| e  ·   notes                —                IDLE             12m04s |
+----------------------------------------------------------------------+
+--[ detail :: 3c24eeee ]----------------------------------------------+
|       cwd : /Users/you/code/todo-app                                  |
|   session : 3c24eeee-ac77-49b8-a31d-f0349acbc48b                      |
| last user : add a hover state to the cards                            |
| last asst : I added a hover state with a subtle scale transform...    |
+----------------------------------------------------------------------+
 [j/k]move [f]ocus [enter]detail [o]pen [c]opy [K]ill [r]efresh [q]uit
```

## Features

- **One screen for every Claude Code session** running on your machine, sorted by attention required (waiting → stuck → working → idle → zombie → ended).
- **Accurate state detection** via Claude Code lifecycle hooks — knows when a session is awaiting a permission prompt vs. actually working.
- **`f`ocus** — jump to the iTerm2 or Terminal.app tab running any session.
- **Zombie cleanup** — surface `claude` processes that have been idle for hours and `K`ill them with one keystroke.
- **Detail view** — peek at the last user/assistant message and the transcript tail without leaving the TUI.
- **Single-file, stdlib only.** No dependencies, no virtualenv. ~1k lines of Python 3.

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
chelin-mission-control            # launch the TUI
chelin-mission-control --setup    # install Claude Code hooks (asked on first launch)
chelin-mission-control --teardown # remove the hooks
```

On first launch, the TUI offers to install lifecycle hooks in `~/.claude/settings.json` for accurate status detection. Pick `Y` (recommended), `n` to skip for this run, or `d` to never ask again. You can always run `--setup` / `--teardown` manually later.

Without the hooks, the TUI still works but uses transcript heuristics, which are less accurate — particularly for "awaiting permission" detection.

## Requirements

- macOS (uses `lsof`, `pbcopy`, and AppleScript for tab focusing)
- Python 3.9+
- [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) installed and writing transcripts to `~/.claude/projects/`
- iTerm2 *or* Terminal.app for the `[f]ocus tab` feature (everything else works without it)

Linux support is plausible but untested — only the focus/clipboard/finder integrations are macOS-specific.

## Keybindings

| key                  | action                                                   |
|----------------------|----------------------------------------------------------|
| `j` / `k` / arrows   | move cursor                                              |
| `g` / `G`            | jump to top / bottom                                     |
| `enter`              | show transcript tail in a modal                          |
| `f`                  | focus the iTerm/Terminal tab running this session        |
| `o`                  | open the project folder in Finder                        |
| `c`                  | copy the session ID to clipboard                         |
| `K`                  | send SIGTERM to the session's process (cleans up zombies)|
| `r`                  | force refresh                                            |
| `q` / `esc`          | quit                                                     |

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
- **Tab focusing.** Gets the TTY of the `claude` PID via `ps -o tty=`, then runs an AppleScript that walks every iTerm2 (or Terminal.app) tab looking for a matching `tty` and activates it.

## Privacy

The script reads transcript files containing the full text of your past Claude Code conversations. **Everything stays local** — there is no network code in this project. The detail pane and transcript modal will display real conversation snippets, so use a fresh test session if you make a screencast.

When `--setup` is enabled, the hook handler writes one JSON line per Claude Code lifecycle event to `~/.claude/mission-control/<session_id>.state.jsonl`. The recorded fields are: timestamp, event name, tool name, tool-use id, permission mode, and the notification matcher — **no message content, prompt text, or tool input/output**.

## Limitations

- The `[f]ocus tab` feature only works for sessions running directly in an iTerm/Terminal tab. Sessions inside `tmux`/`screen` will focus the right outer tab but you'll still need to switch panes manually.
- Without `--setup`, "awaiting permission" detection is a heuristic and frequently misclassifies sessions.
- Sessions that started before `--setup` ran will use the heuristic until they're restarted (no historical hook events to read).
- The cwd → project-dir encoding (slashes become dashes) is ambiguous when paths contain real `-` characters; the script disambiguates by probing the filesystem.

## Contributing

Issues and pull requests welcome at [github.com/chelinho139/chelin-mission-control](https://github.com/chelinho139/chelin-mission-control). Please include your macOS and Python versions, and a brief description of what you saw vs. expected.

## License

[MIT](LICENSE) © Chelinho
