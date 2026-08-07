# dotfiles-claude

Version-controlled global Claude Code config (skills, rules, etc.), symlinked into
`~/.claude/` so every project on this machine picks it up automatically.

## Layout

- `CLAUDE.md` — global rules, symlinked to `~/.claude/CLAUDE.md`
- `rules/` — global rule files, symlinked to `~/.claude/rules`
- `skills/` — global skills, symlinked to `~/.claude/skills`
- `agents/` — global subagents, symlinked to `~/.claude/agents`

## Setup

On a new machine, clone the repo and link each entry into `~/.claude/`:

```bash
REPO="$HOME/Dropbox/project/dotfiles-claude"
git clone https://github.com/pokaa3a/dotfiles-claude.git "$REPO"

mkdir -p "$HOME/.claude"
for f in CLAUDE.md rules skills agents; do
  ln -sfn "$REPO/$f" "$HOME/.claude/$f"
done
```

Adjust `REPO` if the repo lives elsewhere — the symlinks store whatever path is
given here, so it must be the real location.

The `-n` in `ln -sfn` matters. Without it, linking a directory whose name already
exists as a symlink creates a nested link inside it (`~/.claude/skills/skills`)
instead of replacing it. If a path exists as a real directory rather than a
symlink, `ln` refuses to touch it; move it aside first.

Verify:

```bash
ls -l ~/.claude
```

Each of the four entries should show an arrow pointing into the repo. Restart
Claude Code afterwards so it picks up the new directories.
