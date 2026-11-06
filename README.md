# dotfiles

Personal command-line tools, shared across machines.

## Install

```
git clone https://github.com/arianeguay/dotfiles.git ~/dotfiles
mkdir -p ~/.local/bin
ln -sfn ~/dotfiles/bin/pull-all ~/.local/bin/pull-all
ln -sfn ~/dotfiles/bin/claude-context ~/.local/bin/claude-context
```

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
transcripts in `~/.claude/projects`, by day, by ISO week in Montreal time, and by repo
(worktrees count toward their repo). Each
transcript is classed as a conversation, a subagent, or background work (conversation
summaries and subagent warmups), so the session counts are not inflated by the latter.

```
claude-context                       # last 4 weeks, by day and week
claude-context --weeks 2 --sessions  # also list each conversation by title
claude-context --csv ~/context.csv   # one row per transcript, for a pivot table
```
