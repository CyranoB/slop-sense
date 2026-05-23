# Slop Sense

[![skills.sh](https://skills.sh/b/CyranoB/slop-sense)](https://skills.sh/CyranoB/slop-sense)

A skill that detects and rewrites AI-generated text. Paste text in, get back an analysis of which AI patterns it contains, and a rewritten version that sounds human.

## What it does

1. Runs the [slop-detector](https://github.com/CyranoB/slop-detector) algorithmic scorer via `npx` (no install needed, just Node.js). Returns a 0-100 SLOP score with specific word hits, trigram matches, and contrast patterns found.
2. Scans for 33 qualitative AI writing patterns: significance inflation, promotional language, AI vocabulary, copula avoidance, em dash overuse, sycophantic tone, invented concept labels, rhetorical Q&A, false vulnerability, and more.
3. Rewrites the text with a two-pass process: draft, then an anti-AI audit that catches what the first pass missed.
4. **ai;dr mode**: extracts the probable prompt that generated a piece of AI text, with an inflation ratio showing how many words the AI used to say something simple.

The scorer is optional. Without Node.js, the skill still does the full qualitative analysis and rewrite.

Accepts pasted text, URLs (fetches and analyzes the page), or file paths.

## Quickstart

Install the skill with one command. It works for 50+ coding agents:

```bash
npx skills@latest add CyranoB/slop-sense
```

The installer detects your agents, asks which to install for, and places the skill
in the right location.

Common variations:

```bash
npx skills@latest add CyranoB/slop-sense --list
npx skills@latest add CyranoB/slop-sense -a claude-code -y
npx skills@latest add CyranoB/slop-sense -a claude-code -g
```

## Install For Your Coding Tool

Install as a skill/plugin when your agent supports them. The Quickstart command
above is the cross-agent path; per-agent details follow for anyone who wants the
specifics.

<details>
<summary><strong>Claude Code</strong></summary>

Install from the plugin marketplace:

```
/plugin marketplace add CyranoB/slop-sense
/plugin install slop-sense@slop-sense
```

Or install as a skill via the cross-agent command:

```bash
npx skills@latest add CyranoB/slop-sense -a claude-code
```

Install globally instead:

```bash
npx skills@latest add CyranoB/slop-sense -a claude-code -g
```

</details>

<details>
<summary><strong>Codex CLI</strong></summary>

Install the skill for the current project:

```bash
npx skills@latest add CyranoB/slop-sense -a codex
```

Install globally instead:

```bash
npx skills@latest add CyranoB/slop-sense -a codex -g
```

</details>

<details>
<summary><strong>Gemini CLI</strong></summary>

Install the skill for the current project:

```bash
npx skills@latest add CyranoB/slop-sense -a gemini-cli
```

Install globally instead:

```bash
npx skills@latest add CyranoB/slop-sense -a gemini-cli -g
```

</details>

<details>
<summary><strong>Pi Coding Agent</strong></summary>

Install the skill for the current project:

```bash
npx skills@latest add CyranoB/slop-sense -a pi
```

Install globally instead:

```bash
npx skills@latest add CyranoB/slop-sense -a pi -g
```

</details>

<details>
<summary><strong>Kiro CLI</strong></summary>

Install the skill for the current workspace:

```bash
npx skills@latest add CyranoB/slop-sense -a kiro-cli
```

Install globally instead:

```bash
npx skills@latest add CyranoB/slop-sense -a kiro-cli -g
```

</details>

<details>
<summary><strong>Other Agent Skills-compatible tools</strong></summary>

```bash
npx skills@latest add CyranoB/slop-sense
```

Direct skill URLs also work:

```bash
npx skills@latest add https://github.com/CyranoB/slop-sense/tree/main/skills/slop-sense
```

</details>

After installing, ask your agent to check any text for AI patterns. The skill
triggers automatically.

## Score interpretation

| Score | Meaning |
|-------|---------|
| 0-20 | Reads like a human wrote it |
| 20-40 | Mostly human, some AI characteristics |
| 40-60 | Could go either way |
| 60-80 | Probably AI-generated |
| 80-100 | Almost certainly AI-generated |

Anything above 30 is worth a second look.

## The 33 patterns

The skill checks for these AI writing tells, grouped by category:

**Content** (1-7): significance inflation, notability name-dropping, superficial -ing analyses, promotional language, vague attributions, formulaic challenges sections, invented concept labels

**Language** (8-16): AI vocabulary overuse, copula avoidance ("serves as" instead of "is"), negative parallelisms, rule of three, synonym cycling, false ranges, anaphora abuse, "not X. not Y. just Z." countdown, rhetorical Q&A

**Style** (17-22): em dash overuse, boldface overuse, inline-header lists, Title Case headings, emojis in structure, curly quotes

**Communication** (23-29): chatbot artifacts, knowledge-cutoff disclaimers, sycophantic tone, "here's the kicker" false suspense, "think of it as..." patronizing analogies, "imagine a world where..." futurism, false vulnerability

**Filler** (30-33): filler phrases, excessive hedging, "the truth is simple" assertions, generic positive conclusions

Based on [Wikipedia: Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), the [EQBench SLOP score](https://eqbench.com/slop-score.html) methodology, and [tropes.fyi](https://tropes.fyi/).

## Credits

Algorithmic scorer: [slop-detector](https://github.com/CyranoB/slop-detector), built on [slop-score](https://github.com/sam-paech/slop-score) by Samuel J. Paech.

AI writing patterns: [WikiProject AI Cleanup](https://en.wikipedia.org/wiki/Wikipedia:WikiProject_AI_Cleanup).

Additional tropes: [tropes.fyi](https://tropes.fyi/) by Ossama.

Humanizer methodology: [blader/humanizer](https://github.com/blader/humanizer).

## License

MIT
