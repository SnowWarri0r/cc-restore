---
description: Reopen Claude Code sessions that were closed with the terminal, one tab each
argument-hint: "[all | 1 3 | -2 4]"
allowed-tools: Bash(${CLAUDE_PLUGIN_ROOT}/bin/cc-restore:*)
---

Sessions that died with their terminal (tab or window closed, terminal quit, reboot):

!`"${CLAUDE_PLUGIN_ROOT}/bin/cc-restore" --list`

Requested selection: "$ARGUMENTS"

- If the list says "Nothing to restore.", tell the user there is nothing to reopen and stop.
- If a selection was requested, run `"${CLAUDE_PLUGIN_ROOT}/bin/cc-restore" --pick "<selection>"` with it.
- Otherwise show the list to the user as is and ask which to reopen: all of them,
  only some numbers (e.g. `1 3`, the rest stay listed), or all but some (e.g. `-2 4`,
  which also removes those from the list). Then run
  `"${CLAUDE_PLUGIN_ROOT}/bin/cc-restore" --pick "<their answer>"`.

Report the command's output in one line. Each reopened session starts in its own tab
and may stop at the "trust this folder" prompt until the user answers it there.
