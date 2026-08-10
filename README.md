# Canario for Claude Code

A [Claude Code](https://claude.com/claude-code) plugin that teaches Claude to
drive [Canario](https://github.com/canarioterm/releases): list terminals and
panes, read a pane's screen, type into it, split, open terminals in spaces or
fresh git worktrees, and wait for a coding agent in another pane to go idle or
get blocked.

```
/plugin marketplace add canarioterm/claude-plugin
/plugin install canario@canarioterm
```

The skill talks to the app through `canario-cli` and its control socket, and
does nothing when Canario is not installed or not running. The first `send`
of each app run asks for your permission inside Canario, since any local
process can reach the socket.

Running `canario-cli install-cli` from an installed Canario.app links the same
skill into `~/.claude/skills/` directly, so app users get it without this
plugin; the two routes carry identical content.

This repository is a mirror: the skill is maintained in Canario's source
repository and published here by its release pipeline, so the plugin version
always matches the newest app release. Issues about the skill's behavior
belong on [canarioterm/releases](https://github.com/canarioterm/releases/issues).
