# Slop Sense

A Claude Code plugin / cross-agent skill that detects AI writing patterns and rewrites text. The skill itself lives at `skills/slop-sense/SKILL.md`; everything else is packaging.

## Structure

- `skills/slop-sense/SKILL.md` — the skill (33 patterns, workflow, ai;dr mode). Edit this for behavior changes.
- `skills/slop-sense/scripts/score.sh` — optional algorithmic scorer wrapper around `npx slop-detector`. Falls back to qualitative-only if Node is missing.
- `.claude-plugin/plugin.json` — Claude Code plugin manifest. Lists the skill in `skills[]` so `npx skills@latest` can enumerate it.
- `.claude-plugin/marketplace.json` — marketplace listing. Duplicates `version` and metadata from `plugin.json`.
- `scripts/link-skills.sh` — dogfooding helper; symlinks `skills/slop-sense` into `~/.claude/skills` (override with `SKILLS_DEST`).

## When making changes

- **Bump `version` in BOTH `plugin.json` and `marketplace.json`** for any user-visible change. Claude Code uses `version` as the update cache key — installs report "already latest" without a bump.
- The skill is distributed two ways: marketplace install (Claude Code only) and `npx skills@latest add CyranoB/slop-sense` (50+ agents). Don't add Claude-specific paths in `SKILL.md` — it has to work cross-agent.
- The scorer is optional by design. Don't make `score.sh` a hard dependency; the skill must still do useful qualitative analysis when Node is unavailable.
- No build, no tests, no lint. Verification = read the diff and try the skill in a session.

## Release flow

1. Edit `SKILL.md` / scripts.
2. Bump `version` in `plugin.json` AND `marketplace.json` (same value).
3. Commit, PR to `main`. The marketplace picks up the new version on push.
