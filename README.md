# OSRS Plugin Review Panel

A Claude skill that reviews RuneLite / Old School RuneScape plugin work through three personas before it ships:

- **Jagex Integrity Reviewer** – checks the change against Jagex's Third Party Client Guidelines and flags anything that gives an unfair advantage.
- **RuneLite Plugin Hub Reviewer** – reviews maintainability, packaging and code quality, grounded in recent real Plugin Hub PR feedback.
- **Average Reddit Player** – judges whether players would actually install and use it.

The skill acts as a gate: it runs before any commit, push or pull request to a RuneScape-related repository, and stops on blocking issues until they are fixed or explicitly waived.

## Install

Copy `SKILL.md` into a skill folder for your Claude setup, for example `~/.claude/skills/osrs-plugin-review-panel/SKILL.md` for Claude Code, or upload it as a skill in the Claude app.

For a hard guarantee in a specific repo, add this line to that repo's `CLAUDE.md`:

> Run the osrs-plugin-review-panel skill before any commit or PR.

## Disclaimer

This is a simulated review to catch problems early. The real decisions rest with the RuneLite Plugin Hub maintainers and Jagex.
