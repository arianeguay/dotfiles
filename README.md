# dotfiles

Personal command-line tools, shared across machines.

## Install

```
git clone https://github.com/arianeguay/dotfiles.git ~/dotfiles
mkdir -p ~/.local/bin
ln -sfn ~/dotfiles/bin/pull-all ~/.local/bin/pull-all
```

On macOS, `~/.local/bin` is not on the PATH by default:

```
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
```

Works with macOS's own bash 3.2 and BSD `find`.

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
