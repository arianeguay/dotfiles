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

## athena-run

Runs a command on athena from the current git repository, so heavy jobs (a test suite that
loads torch, a render, a build) never run on the laptop.

```
athena-run 'bun install --frozen-lockfile && bun test lib/house3d'
athena-run --fetch report.json 'python3 scripts/measure.py > report.json'   # copy it back
```

Each run syncs the repo's git file list (tracked files, plus untracked ones git does not
ignore) to `athena:athena-run/<repo>/<branch>/`. A file git ignores, such as `node_modules`
or a `.venv`, stays there between runs; any other file missing from the list is deleted,
so a file removed on the branch is gone on athena too. The command runs in the same
subdirectory, through `bash -lc` with its arguments joined as ssh joins them, and the exit
code is the command's. `--fetch <path>` (relative to that subdirectory, repeatable) copies
a file or directory back afterwards, even when the command failed.

Jobs run one at a time behind `flock ~/athena-run/lock` (a second one prints that it is
waiting), at `nice 19` and idle IO priority, so the LLMs athena serves keep answering. An
athena that does not answer within 5 s is reported as such: it may be powered off.

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
claude-context --studio-json ~/studio.json      # Studio's own counts per week, repo and model
```

Both JSON files hold counts only: no prompt text, titles or file content. Keep them apart: a Studio call can be in both.

Reading the report:

- **Pace** cuts every week at the time elapsed in the current one. Comparing a running week to a full one always makes it look low.
- **Weight** applies public prices to compare periods. The limit's real formula is not public.
- **w5m and w1h** are cache writes by lifetime. Only Claude models count: a local model reached through `ANTHROPIC_BASE_URL` is named and left out of the tables.
- **studio** counts sessions Studio's Claude provider spawned, which otherwise read as conversations under the parent's entrypoint.
- **Images** are estimated as width x height / 750. They are read back on each call until a compaction, which costs far more than sending them once. On one repo the real context jump ran about 1.22 times this estimate.
- **Studio's own records** come from `.studio/runs` anywhere under the home directory, shown by week beside the transcripts of Studio sessions. Compare the totals, not repo by repo: a run's record and its transcripts can sit under different repos. On 2026-11-06 they agreed except for 268 Haiku entries with no transcript, about 6 M tokens. A pipeline in a container keeps its records there, out of reach, so only its transcripts show (skynet-counter on apollon).
