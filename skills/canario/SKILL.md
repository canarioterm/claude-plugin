---
name: canario
description: "Control Canario, a macOS terminal with agent-aware sessions. Use only when the user explicitly mentions Canario or asks to inspect or control its terminals, panes, or agents. Do not use merely because a task could benefit from a background terminal or parallel work."
---

# Canario

Canario organizes terminals into spaces and panes, detects coding agents
running inside them (working / blocked / idle), and exposes the current
session through a control socket and the `canario-cli` binary.

Before issuing any control command, verify Canario is running and the
socket exists:

```bash
CLI="/Applications/Canario.app/Contents/MacOS/canario-cli"
test -x "$CLI" && test -S "$HOME/Library/Application Support/canario/control.sock"
```

If the check fails, say that Canario is not available and stop.

## Commands

```bash
"$CLI" list
```

Every terminal and pane, with short ids, the running process, and the
detected agent state. Pane ids (8-hex prefixes) address the other
commands.

```bash
"$CLI" read --panel <id> --lines 40      # a pane's visible screen
"$CLI" send --panel <id> "npm test"      # type + run in a pane
"$CLI" send --panel <id> --raw "y"       # type without newline
"$CLI" split [--down]                    # split the focused terminal
"$CLI" new --space Work --cwd ~/api      # new terminal in a space
"$CLI" new --worktree                    # new git worktree + terminal
"$CLI" wait --panel <id> --until blocked --timeout 300
"$CLI" notify                            # ask for the user's attention
"$CLI" install-cli                       # symlink as `canario` on PATH
```

`new --worktree` creates a git worktree on a fresh branch beside the
repository (`<repo>-worktrees/<branch>`) and opens a terminal in it, so
parallel agents do not fight over one checkout. It prints
`branch<TAB>path` before the panel id.

The first `send` of each app run asks the user to allow it, since any
local process can reach the socket. Expect a prompt, and do not retry in
a loop if it is denied.

`wait` blocks until the pane's detected agent state matches
(`blocked` | `working` | `idle`); exit code 2 means timeout, and the
final state is printed either way.

## Typical patterns

Watch a sibling agent and take over when it needs input:

```bash
"$CLI" wait --panel 9d6bcc93 --until blocked && "$CLI" read --panel 9d6bcc93 --lines 20
```

Fan work out to a new pane and collect the result:

```bash
panel=$("$CLI" split --down)
"$CLI" send --panel "$panel" "cargo test 2>&1 | tail -5"
sleep 30 && "$CLI" read --panel "$panel" --lines 10
```

## Wire protocol

The CLI is a thin client over a unix socket at
`~/Library/Application Support/canario/control.sock`; talk to it directly
when a shell-out is awkward. Two formats, chosen by the first byte of the
connection:

- **JSON lines** (first byte `{`): one JSON object per line, replies are
  newline-terminated JSON.
- **MessagePack** (anything else): each message is a 4-byte big-endian
  length prefix followed by a MessagePack map; replies use the same
  framing.

Request keys mirror the CLI: `method`, `panel`, `terminal`, `direction`,
`space`, `cwd`, `text`, `lines`, `worktree`, `until`, `timeout`. Errors
come back as `{"error": "..."}`.

## Cautions

- `send` types into a live terminal the user can see; never send
  destructive commands without the user's explicit request.
- `read` returns whatever is on screen, which may include secrets the
  user has displayed; do not exfiltrate it.
- Pane ids change across app restarts; always start from `list`.
- Worktrees are removed only with the user's confirmation, and never
  when they contain uncommitted work.
