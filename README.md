# dotfiles

Personal command-line tools, shared across machines.

## Install

```
git clone https://github.com/arianeguay/dotfiles.git ~/dotfiles
~/dotfiles/bin/pull-all ~/dotfiles
```

Every run links each tool in `bin/` into `~/.local/bin`, so a new tool reaches a machine with
its next pull. A file or link already at that path is left alone.

On macOS, `~/.local/bin` is not on the PATH by default:

```
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

`pull-all` works with macOS's own bash 3.2 and BSD `find`. `claude-context` needs Python 3.9 or later.

## pull-all

Fast-forwards every git repository under `~/dev`, `~/src` and `~/dotfiles`, or under
the directories given as arguments. It never stashes, merges or forces: a repository
with local changes, unpushed commits or a diverged branch is reported and left alone.

Each repository has a target branch: `git config pull-all.target <branch>` when set,
otherwise the remote's default branch. A checkout on another branch is reported and
not pulled. In a terminal, a clean one is offered a switch; under cron it never prompts.

```
pull-all                    # default directories
pull-all ~/work ~/dotfiles  # explicit directories
git -C ~/work/app config pull-all.target develop
```

## claude-context

Context size and token usage of every Claude Code session on the machine, read from the
transcripts in `~/.claude/projects`. Tokens are split into fresh input, cache write, cache
read and output, and each API call is counted once (the last record per message and request,
across transcripts) on its own 5-minute bucket, in Montreal time. Reported by day, by week,
and by repo (worktrees count toward their repo). Each transcript is classed as a conversation,
a subagent, or background work (conversation summaries and subagent warmups).

```
claude-context                                  # last 4 weeks, by day, week (ending Sunday 18:00) and repo
claude-context --reset 2026-11-09T01:59         # weeks of 7 days ending Monday 01:59 instead
claude-context --weeks 2 --sessions             # also list each conversation by title
claude-context --csv ~/context.csv              # one row per transcript, for a pivot table
claude-context --json ~/context.json            # counts per host, bucket, cwd, model and kind
```

The JSON holds counts only: no prompt text, titles or file content.
