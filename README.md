# cc-restore

Reopen every Claude Code session a restart killed — one terminal tab each, back in
its original directory, conversation intact.

```
$ cc-restore
  1  10-08 16:24  ~/work/api                Fix flaky integration test
  2  10-08 15:51  ~/work/web                Migrate settings page to new form lib
  3  10-08 11:02  ~                         Weekly report draft

Enter = all | 1 3 = only these | -2 4 = all but these | q = quit:
Reopened 3 session(s).
```

If you keep a dozen `claude` sessions open across tabs, a terminal crash, an OS update
or a reboot means hunting each one down in `claude --resume` by hand. cc-restore keeps
track of them for you.

## How it works

- A `SessionStart` hook records each interactive session (id, working directory,
  transcript) in `~/.claude/cc-restore/live/`.
- A `SessionEnd` hook removes the record when you leave on purpose — `/exit`,
  Ctrl-D, double Ctrl-C, `/clear`.
- A session that dies with its terminal (tab or window closed, terminal quit, machine
  reboot) keeps its record. The next `claude` you start tells you how many are
  waiting, and `/cc-restore:restore` (or `cc-restore` in a shell) reopens the ones
  you pick with `claude --resume <id>`. A resumed session keeps its id and registers
  itself again.

Titles come from your `/rename` name, then Claude's auto title, then the first prompt.
Name your sessions with `/rename` and the list reads at a glance.

## Install

Requires Python 3.8+ (standard library only).

### As a Claude Code plugin (recommended)

```
/plugin marketplace add SnowWarri0r/cc-restore
/plugin install cc-restore@cc-restore
```

The hooks come with the plugin; nothing in your settings is edited. To also have the
`cc-restore` shell command, add the stable link the plugin maintains to your `PATH`:

```sh
export PATH="$HOME/.claude/cc-restore/bin:$PATH"
```

### Standalone

```sh
git clone https://github.com/SnowWarri0r/cc-restore.git ~/cc-restore
ln -s ~/cc-restore/bin/cc-restore ~/.local/bin/cc-restore   # any directory on your PATH
cc-restore install
```

`install` adds the two hooks to `~/.claude/settings.json` (a timestamped backup is
written next to it) and registers the sessions that are already running, so the very
next restart is covered. `cc-restore uninstall` removes the hooks. Pick one of the two
install methods, not both. `CLAUDE_CONFIG_DIR` is respected.

## Usage

Inside Claude Code:

```
/cc-restore:restore          # show the list and ask which to reopen
/cc-restore:restore all
/cc-restore:restore 1 3      # only these
/cc-restore:restore -2       # all but these
```

From a shell:

```sh
cc-restore              # list killed sessions, pick which to reopen
cc-restore --all        # reopen all without asking
cc-restore --pick "1 3" # same choices as above, non-interactive
cc-restore --list       # only list
cc-restore --days 30    # look further back (default: 7 days)
cc-restore --terminal tmux   # override terminal detection
```

Sessions you skip are dropped from the list; their transcripts stay, so
`claude --resume` still finds them.

## Terminals

| Terminal | How sessions reopen | Detected by |
|---|---|---|
| Ghostty 1.3+ (macOS) | new tab in the front window | `TERM_PROGRAM=ghostty` |
| iTerm2 (macOS) | new tab in the current window | `TERM_PROGRAM=iTerm.app` |
| tmux (any OS) | new window per session, named after it | `$TMUX` |
| anything else | prints one `cd … && claude --resume …` line per session | fallback |

On macOS the first run asks for permission to control the terminal app
(System Settings → Privacy & Security → Automation).

## Notes

- Claude Code asks "Do you trust this folder?" when a directory has not been trusted
  permanently (your home directory, for one). Restored tabs in such a directory stop
  at that prompt until you answer it.
- Headless runs (`claude -p`) are not recorded.
- Sessions that never got a message have no transcript and are skipped.

## License

MIT
