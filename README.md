# Skills Database

General-purpose Claude Code skills that work in any project. No project-specific or personal details.

## Skills

| Skill | What it does |
|---|---|
| `adversarial-review` | Read-only, skeptical review of a branch or PR diff |
| `review-pr` | Reviews a contributor's PR and posts a pending GitHub review with inline comments |
| `triage-issue` | Triages a GitHub issue: duplicates, missing info, labels |

## Install on a new device

```bash
git clone https://github.com/nickybmon/Skills-Database.git ~/Documents/Github/Skills-Database
mkdir -p ~/.claude/skills
for d in ~/Documents/Github/Skills-Database/skills/*/; do ln -sfn "$d" ~/.claude/skills/$(basename "$d"); done
```

Run the loop again after pulling new skills.
