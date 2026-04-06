# Slop Sense

A Claude Code plugin that detects and rewrites AI-generated text. Paste text in, get back an analysis of which AI patterns it contains, and a rewritten version that sounds human.

## What it does

1. Runs the [slop-detector](https://github.com/CyranoB/slop-detector) algorithmic scorer via `npx` (no install needed, just Node.js). Returns a 0-100 SLOP score with specific word hits, trigram matches, and contrast patterns found.
2. Scans for 33 qualitative AI writing patterns: significance inflation, promotional language, AI vocabulary, copula avoidance, em dash overuse, sycophantic tone, invented concept labels, rhetorical Q&A, false vulnerability, and more.
3. Rewrites the text with a two-pass process: draft, then an anti-AI audit that catches what the first pass missed.
4. **ai;dr mode**: extracts the probable prompt that generated a piece of AI text, with an inflation ratio showing how many words the AI used to say something simple.

The scorer is optional. Without Node.js, the skill still does the full qualitative analysis and rewrite.

Accepts pasted text, URLs (fetches and analyzes the page), or file paths.

## Install

### From the marketplace

```
/plugin marketplace add CyranoB/slop-sense
/plugin install slop-sense@slop-sense
```

### Direct install

```
claude --plugin-dir /path/to/slop-sense
```

### Manual

```bash
git clone https://github.com/CyranoB/slop-sense.git ~/.claude/skills/slop-sense
```

Then ask Claude to check any text for AI patterns. The skill triggers automatically.

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
