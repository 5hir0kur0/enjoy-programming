# enjoy-programming

An [Agent Skill](https://agentskills.io) for pair programming where the human
writes the code and the agent plans, researches, briefs, tests, and reviews.
It's based on
[How to keep enjoying programming in a world of LLMs](https://discourse.haskell.org/t/how-to-keep-enjoying-programming-in-a-world-of-llms/14705),
with the working style of
[Superpowers](https://github.com/obra/superpowers) (TDD, asking questions,
planning, review cycles) and the anti-overengineering ladder of [Ponytail](https://github.com/DietrichGebert/ponytail).

The skill lives in [`skills/enjoy-programming/`](skills/enjoy-programming/SKILL.md).

## Install

Skills are plain directories with a `SKILL.md`, so installing means putting
(or symlinking) that directory where your tool looks for skills. Symlinks
keep every tool in sync with this repo:

```sh
git clone <this repo> ~/some/path
SKILL=~/some/path/skills/enjoy-programming
```

| Tool | Personal (all projects) | Per project |
|------|-------------------------|-------------|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Zed | `~/.agents/skills/` | `.agents/skills/` |
| VS Code (Copilot) | `~/.agents/skills/`, `~/.claude/skills/` or `~/.copilot/skills/` | `.agents/skills/`, `.claude/skills/` or `.github/skills/` |
| Codex, Gemini CLI, Cursor, others | usually `~/.agents/skills/`, check the tool's docs | usually `.agents/skills/` |

For all of them at once:

```sh
mkdir -p ~/.agents/skills ~/.claude/skills
ln -s "$SKILL" ~/.agents/skills/enjoy-programming   # Zed, VS Code, Codex, ...
ln -s "$SKILL" ~/.claude/skills/enjoy-programming   # Claude Code
```

For a single project, link or copy the directory into that project's
`.agents/skills/` (plus `.claude/skills/` for Claude Code) instead.

## Use

Invoke it explicitly (`/enjoy-programming` in Claude Code, Zed, and VS Code),
or let the agent pick it up from its description. It works best as the
only process skill that's active. Skills that tell the agent to work
autonomously (for example Superpowers' `subagent-driven-development`)
contradict it.

Plans go to `docs/plans/` and research notes to `docs/research/`, unless
the project says otherwise.
