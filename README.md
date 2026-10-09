# cc-restore

Restarted your machine with a dozen Claude Code sessions open? Get them all back in
one go — each in its own terminal tab, in its original directory, with the
conversation intact.

```
$ claude
  ⎿  cc-restore: 3 Claude Code sessions closed with the terminal.
     Run /cc-restore:restore to reopen them.

> /cc-restore:restore
  1  10-08 16:24  ~/work/api       Fix flaky integration test
  2  10-08 15:51  ~/work/web       Migrate settings page to new form lib
  3  10-08 11:02  ~                Weekly report draft

  Reopen all of them, or pick some (e.g. 1 3, or -2 for all but #2)?

> all
  Reopened 3 session(s).
```

## Why not just `claude --resume`?

cc-restore calls `claude --resume` under the hood. What it adds is the bookkeeping
around it:

| | `claude --resume` | cc-restore |
|---|---|---|
| What it lists | every past session, ones you finished and ones that were cut off alike | only sessions that were still open when the terminal went away |
| Restoring ten sessions | ten rounds of: open a tab, `cd`, `claude --resume`, find the right one | one command, one tab per session |
| Working directory | yours to get right (`claude -c` only picks the latest in the current directory) | each session reopens where it was |
| After a restart | you have to remember what was open | the first new session tells you what is waiting |

It is not a replacement for `--resume`: to dig up a session you closed on purpose
last month, use `--resume`.

## How it works

- A `SessionStart` hook records every interactive session — id, working directory,
  transcript path, process id — in `~/.claude/cc-restore/live/`.
- A `SessionEnd` hook removes the record when you leave on purpose: `/exit`, Ctrl-D,
  double Ctrl-C, `/clear`.
- A session that dies with its terminal — tab or window closed, terminal quit,
  machine rebooted — keeps its record.
- Restoring lists the records whose process is gone and reopens the ones you pick
  with `claude --resume <id>`. A reopened session keeps its id and records itself
  again, so the cycle repeats.

Each entry is titled with your `/rename` name, else Claude's auto-generated title,
else the first prompt. Naming sessions with `/rename` makes the list easy to scan.

## Install

Requires Python 3.8+ (standard library only) and macOS or Linux.

### As a Claude Code plugin (recommended)

In Claude Code:

```
/plugin marketplace add SnowWarri0r/cc-restore
/plugin install cc-restore@cc-restore
```

The hooks ship with the plugin; your settings files are not touched. Sessions you
start from now on are tracked. Sessions that were already open when you installed
are tracked from the next time they start.

To also run `cc-restore` from a shell, put the link the plugin keeps up to date on
your `PATH`:

```sh
export PATH="$HOME/.claude/cc-restore/bin:$PATH"
```

### Standalone

```sh
git clone https://github.com/SnowWarri0r/cc-restore.git ~/cc-restore
ln -s ~/cc-restore/bin/cc-restore ~/.local/bin/cc-restore   # any directory on your PATH
cc-restore install
```

`install` adds the two hooks to `~/.claude/settings.json`, writing a timestamped
backup next to it first, and records the sessions that are already running so the
very next restart is covered.

Use one install method, not both. `CLAUDE_CONFIG_DIR` is respected either way.

## Usage

Inside Claude Code:

```
/cc-restore:restore          # show the list, then ask which to reopen
/cc-restore:restore all      # reopen everything
/cc-restore:restore 1 3      # only #1 and #3; the rest stay listed
/cc-restore:restore -2       # everything except #2, and forget #2
```

From a shell:

```sh
cc-restore                   # show the list, then ask
cc-restore --all             # reopen everything
cc-restore --pick "1 3"      # same choices as above, without the prompt
cc-restore --list            # just show the list
cc-restore --days 30         # look back further (default: 7 days)
cc-restore --terminal tmux   # override terminal detection
```

Only sessions you exclude with `-N` leave the list; anything you simply did not pick
stays for next time. Forgetting a session only removes it from this list — its
transcript is untouched, so `claude --resume` still finds it.

## Terminals

| Terminal | Sessions reopen as | Detected by |
|---|---|---|
| Ghostty 1.3+ (macOS) | tabs in the front window | `TERM_PROGRAM=ghostty` |
| iTerm2 (macOS) | tabs in the current window | `TERM_PROGRAM=iTerm.app` |
| tmux (macOS, Linux) | one window per session, named after it | `$TMUX` |
| anything else | printed `cd … && claude --resume …` lines to paste | fallback |

The first time cc-restore drives Ghostty or iTerm2, macOS asks whether to allow it
(System Settings → Privacy & Security → Automation). If the terminal does not
respond, cc-restore stops after 30 seconds and leaves the list as it was.

## Good to know

- **Closing a tab counts as "cut off".** cc-restore cannot tell a tab you closed on
  purpose from one a reboot took down; exclude it with `-N` when you restore and
  it is gone from the list.
- **Trust prompts.** If a directory is not permanently trusted — your home directory,
  for instance — each session reopened there waits at Claude Code's "Do you trust
  this folder?" prompt until you answer it.
- **Not restored:** tab order, splits and window layout, and the terminal's
  scrollback. The conversation itself is fully restored.
- **Skipped automatically:** headless runs (`claude -p`), and sessions that never got
  a message (there is nothing to resume).

## Uninstall

```
/plugin uninstall cc-restore@cc-restore      # plugin
cc-restore uninstall                         # standalone
```

Then remove the session records with `rm -rf ~/.claude/cc-restore`.

## License

MIT
