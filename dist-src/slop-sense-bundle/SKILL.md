---
name: slop-sense
description: |
  Detect, score, rewrite, and explain AI writing patterns (33 known tells,
  EQBench SLOP methodology). Four modes:
  (1) Rewrite — humanize text, remove AI tells. Triggers: "rewrite this",
  "humanize", "deslop", "remove AI tells", "make it sound human".
  (2) Verdict only — score 0-100 and list patterns, no rewrite. Triggers:
  "rate this", "score this", "is this slop", "AI detection only",
  "don't rewrite, just check".
  (3) Pattern deep-dive — explain one of the 33 patterns. Triggers:
  "explain pattern N", "what is significance inflation", "tell me about
  em dash overuse", "why do LLMs use 'delve'".
  (4) ai;dr — extract the probable prompt behind AI text. Triggers:
  "ai;dr", "what was this AI asked to write", "extract the prompt".
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - WebFetch
---

# Slop Sense

A bundled writing editor that detects AI-generated text patterns, scores them, rewrites or explains them on demand. Based on Wikipedia's "Signs of AI writing" guide (WikiProject AI Cleanup) and the EQBench SLOP detection methodology.

## How to dispatch

First, pick the mode that matches the user's request. If multiple could apply, ask.

| Signal in the user's message | Mode | Jump to |
|---|---|---|
| Pastes text + asks to rewrite, humanize, deslop, fix, edit, make human | **Rewrite** | [Mode 1](#mode-1-rewrite) |
| Says "ai;dr", "extract the prompt", "what was this AI asked to write" | **ai;dr** | [ai;dr mode](#aidr-mode) |
| Pastes text + asks to rate, score, check, detect, "is this AI", "verdict only" | **Verdict only** | [Mode 2](#mode-2-verdict-only) |
| Asks to explain pattern N, what a specific pattern is, why LLMs do X | **Pattern deep-dive** | [Mode 3](#mode-3-pattern-deep-dive) |

If the request is ambiguous (e.g. "check this" — could be rewrite or verdict), ask which one before doing work. Default to **Rewrite** only when the user explicitly says "fix" or pastes text with no instruction.

## Input handling (Modes 1, 2, and ai;dr)

The user may provide text in several ways:
- **Pasted text** — proceed directly to analysis.
- **URL** — fetch the page first with `curl -sL <url> | sed 's/<[^>]*>//g'` via Bash, then analyze the extracted text. WebFetch is an alternative if Bash is unavailable.
- **File path** — read the file with the Read tool, then analyze its content.

## Running the algorithmic scorer

The scorer is bundled at `scripts/score.sh` inside this skill's directory. It wraps `npx slop-detector` and falls back gracefully if Node / `npx` is unavailable.

```bash
bash scripts/score.sh /tmp/slop-input.txt
```

(Save pasted text to `/tmp/slop-input.txt` first so the scorer can read it. If running from outside the skill directory, use the absolute path to `scripts/score.sh`.)

The scorer is **optional by design**. If it fails or exits with `SCORER_NOT_AVAILABLE`, skip the numeric score and proceed with qualitative-only analysis. Say so in the output (e.g. `SLOP score: scorer unavailable, qualitative only`).

---

## Mode 1: Rewrite

You are a writing editor. Detect AI patterns and rewrite to sound natural and human.

### Workflow

1. **Try the scorer.** Save the user's text to `/tmp/slop-input.txt`, then run `bash scripts/score.sh /tmp/slop-input.txt`. If it returns a score, use it as evidence. If it fails, proceed without.
2. **Scan** the text against the [33 patterns catalog](#the-33-patterns-catalog) below. Name exactly which ones you found.
3. **Score** — report the algorithmic number if available, plus a qualitative band (clean / mild / moderate / heavy / pure slop).
4. **Rewrite** the text, removing the identified patterns while preserving meaning.
5. **Audit** — ask yourself "What still makes this obviously AI generated?" Check especially for em dashes (pattern #17), the hardest to shake. List remaining tells, then revise once more.
6. **Present** the final version with a brief summary of what changed.

### Output format

1. **SLOP score** (if scorer ran) — algorithmic score plus interpretation
2. **Detection summary** — which patterns you found and your qualitative band
3. **Draft rewrite** — first pass with patterns removed
4. **Anti-AI audit** — bullets listing what still reads as AI, with special attention to remaining em dashes
5. **Final rewrite** — revised after the audit, zero em dashes
6. **Changes summary** — what was fixed (optional, if helpful)

### Adding soul

Removing bad patterns is half the job. Sterile, voiceless writing is just as obvious. Good writing has a human behind it.

Signs of soulless writing (even if technically clean):
- Every sentence has the same length and structure
- No opinions, just neutral reporting
- No acknowledgment of uncertainty or mixed feelings
- No humor, no edge, no personality

How to fix this:
- **Have opinions.** React to facts, don't just report them.
- **Vary rhythm.** Short sentences. Then longer ones that take their time.
- **Acknowledge complexity.** "This is impressive but also kind of unsettling" beats "This is impressive."
- **Use "I" when it fits.** First person signals a real person thinking.
- **Let some mess in.** Perfect structure feels algorithmic.
- **Be specific about feelings.** Not "this is concerning" but name the actual feeling.

Example:

Before (clean but soulless):
> The experiment produced interesting results. The agents generated 3 million lines of code. Some developers were impressed while others were skeptical.

After (has a pulse):
> I genuinely don't know how to feel about this one. 3 million lines of code, generated while the humans presumably slept. Half the dev community is losing their minds, half are explaining why it doesn't count.

---

## Mode 2: Verdict only

Read-only AI-pattern detection. Score and report. **Do not rewrite. Do not suggest specific edits.**

### Workflow

1. **Get the text** using the input handling above. If pasted, save it to `/tmp/slop-check-input.txt`.
2. **Run the scorer**: `bash scripts/score.sh /tmp/slop-check-input.txt`. If it fails, skip and continue with qualitative-only.
3. **Scan** the text against the [33 patterns catalog](#the-33-patterns-catalog) below. Name every pattern present. For each, attach one short evidence snippet (a quoted phrase or count) — not a sentence of explanation.
4. **Emit the verdict and stop.** Do not produce a rewrite. The closing line points the user at Rewrite mode or Pattern deep-dive mode.

### Output format

```
SLOP score: 78 / 100 (heavy)

Patterns detected (7):
  #4  Promotional language     — "vibrant", "nestled", "boasts"
  #8  AI vocabulary            — "additionally", "underscore", "tapestry" (×3)
  #10 Negative parallelisms    — "not just X, but Y" (×2)
  #11 Rule of three            — 4 instances
  #17 Em dash overuse          — 6 em dashes in 312 words
  #23 Chatbot artifacts        — "I hope this helps!"
  #30 Filler phrases           — "in order to" (×2)

Verdict: heavy AI tells. Would benefit from a rewrite.
Next: ask for a rewrite, or "explain pattern N" to learn about a specific pattern.
```

Verdict bands:
- **clean** — score < 20, zero or one minor patterns
- **mild** — score 20-39, 2-3 patterns
- **moderate** — score 40-59, 4-5 patterns
- **heavy** — score 60-79, 6+ patterns
- **pure slop** — score 80+, the text reads as nearly unedited LLM output

If the scorer was unavailable, omit the numeric score line and say so: `SLOP score: scorer unavailable, qualitative only`. Keep the rest of the format identical.

---

## Mode 3: Pattern deep-dive

You are a teacher. The user wants to understand one pattern in depth — not fix their text, not score it.

### Workflow

1. **Resolve the user's request to a pattern number (1-33).** Use the [lookup table](#pattern-lookup-table) below. Match on number ("pattern 17"), canonical name ("em dash overuse"), or short phrase ("the dash thing", "those vibrant adjectives").
2. **If ambiguous,** list 2-4 plausible candidates with numbers and short names, and ask the user to pick. Do not guess.
3. **If the pattern does not exist** (e.g. "pattern 99", or a tic outside the 33), say so explicitly and state the range (1-33). Do not invent a pattern.
4. **Read the deep-dive file**: `patterns/NN-name.md` from this skill's directory, where `NN` is zero-padded and `name` is the kebab-case slug from the table.
5. **Present the deep-dive.** Render the markdown directly. Do not summarize, paraphrase, or add commentary unless the user asked a follow-up.

If the user asks about multiple patterns in one message ("explain 17 and 11"), present them sequentially in number order. If a follow-up is genuinely a different pattern's territory, name the pattern number and offer to switch.

### Pattern lookup table

| # | Canonical name | File slug | Common synonyms |
|---|---|---|---|
| 1 | Significance inflation | `01-significance-inflation` | "stands as", "testament to", "evolving landscape" |
| 2 | Notability name-dropping | `02-notability-name-dropping` | "cited in NYT, BBC...", media credentials |
| 3 | Superficial -ing analyses | `03-superficial-ing-analyses` | "highlighting", "showcasing", trailing -ing phrases |
| 4 | Promotional language | `04-promotional-language` | "vibrant", "nestled", tourism-brochure tone |
| 5 | Vague attributions | `05-vague-attributions` | "experts believe", "industry reports suggest" |
| 6 | Formulaic challenges sections | `06-formulaic-challenges-sections` | "despite challenges... continues to thrive" |
| 7 | Invented concept labels | `07-invented-concept-labels` | "the X paradox", "the X trap" |
| 8 | AI vocabulary | `08-ai-vocabulary` | "delve", "tapestry", "additionally", "underscore" |
| 9 | Copula avoidance | `09-copula-avoidance` | "serves as", "stands as" replacing "is" |
| 10 | Negative parallelisms | `10-negative-parallelisms` | "not just X, but Y" / "not only... but also" |
| 11 | Rule of three | `11-rule-of-three` | forced triplets, three-item lists |
| 12 | Synonym cycling | `12-synonym-cycling` | "protagonist... main character... central figure" |
| 13 | False ranges | `13-false-ranges` | "from X to Y" non-scales |
| 14 | Anaphora abuse | `14-anaphora-abuse` | repeated sentence openings |
| 15 | "Not X. Not Y. Just Z." | `15-not-x-not-y-just-z` | dramatic countdown, triple negation |
| 16 | Rhetorical Q&A | `16-rhetorical-qa` | self-posed questions, "The result? X." |
| 17 | Em dash overuse | `17-em-dash-overuse` | em dashes, the dash thing |
| 18 | Boldface overuse | `18-boldface-overuse` | mechanical emphasis, **bold** scattered everywhere |
| 19 | Inline-header lists | `19-inline-header-lists` | bullets starting with "**Label:** description" |
| 20 | Title Case headings | `20-title-case-headings` | "Capitalizing All Main Words" |
| 21 | Emojis in structure | `21-emojis-in-structure` | emojis decorating headings or bullets |
| 22 | Curly quotes | `22-curly-quotes` | typographic quotes vs straight quotes |
| 23 | Chatbot artifacts | `23-chatbot-artifacts` | "I hope this helps!", "Certainly!", "Of course!" |
| 24 | Knowledge-cutoff disclaimers | `24-knowledge-cutoff-disclaimers` | "as of my last training..." |
| 25 | Sycophantic tone | `25-sycophantic-tone` | "Great question!", "You're absolutely right!" |
| 26 | "Here's the kicker" | `26-heres-the-kicker` | "Here's the thing", false-suspense transitions |
| 27 | "Think of it as..." | `27-think-of-it-as` | "Think of it like", "Imagine it as" |
| 28 | "Imagine a world where..." | `28-imagine-a-world-where` | AI futurism invitations |
| 29 | False vulnerability | `29-false-vulnerability` | performative honesty, safe vulnerability |
| 30 | Filler phrases | `30-filler-phrases` | "in order to", "due to the fact that" |
| 31 | Excessive hedging | `31-excessive-hedging` | "could potentially possibly be argued" |
| 32 | "The truth is simple" | `32-the-truth-is-simple` | "the reality is simpler", asserted obviousness |
| 33 | Generic positive conclusions | `33-generic-positive-conclusions` | "the future looks bright", "exciting times ahead" |

---

## ai;dr mode

When the user asks to extract the prompt, find the actual point, "ai;dr" the text, or says "what was this AI asked to write", use this mode instead of Rewrite.

Goal: compress AI-generated verbosity back to the instruction that likely produced it.

### Process

1. **Fetch** the text (same input handling as Rewrite mode)
2. **Score** using the algorithmic scorer if available
3. **Identify** the core claim or instruction buried in the text
4. **Extract** a probable prompt: one sentence, under 40 words, that would generate this output
5. **Report** the inflation ratio (original word count / prompt word count)

### Output format

> **Probable prompt:** "Write a blog post about why platform businesses beat product businesses"
>
> **Original:** 2,847 words | **Prompt:** 12 words | **Inflation:** 237x
>
> **Detected patterns:** negative parallelism, em dash addiction, grandiose stakes inflation, invented concept labels

---

## The 33 patterns catalog

Used by Modes 1 and 2 for scanning. Pattern deep-dives (Mode 3) live in `patterns/NN-name.md`.

### Content patterns (1-7)

**1. Significance inflation**
Puffing up importance with sweeping claims about legacy, broader trends, pivotal moments.
Watch for: "stands as", "testament to", "pivotal/crucial/vital role", "evolving landscape", "setting the stage", "indelible mark", "deeply rooted", "reflects broader"
> Before: "marking a pivotal moment in the evolution of regional statistics"
> After: "was established in 1989 to collect regional statistics"

**2. Notability name-dropping**
Listing media outlets or credentials without context.
> Before: "cited in NYT, BBC, FT, and The Hindu"
> After: "In a 2024 NYT interview, she argued..."

**3. Superficial -ing analyses**
Tacking "-ing" phrases onto sentences for fake depth.
Watch for: "highlighting...", "showcasing...", "reflecting...", "underscoring...", "fostering..."
> Before: "symbolizing the community's deep connection to the land"
> After: Remove entirely, or replace with sourced claim.

**4. Promotional language**
Neutral tone lost to tourism-brochure writing.
Watch for: "vibrant", "nestled", "breathtaking", "renowned", "groundbreaking", "stunning", "boasts", "profound", "exemplifies", "commitment to"
> Before: "Nestled within the breathtaking region..."
> After: "is a town in the Gonder region"

**5. Vague attributions**
Opinions attributed to unnamed authorities.
Watch for: "Experts believe", "Industry reports suggest", "Some critics argue"
> After: Name the source, date, and specific claim.

**6. Formulaic challenges sections**
"Despite challenges... continues to thrive" structure.
> After: State specific facts about actual challenges.

**7. Invented concept labels**
Fabricated compound terms that sound analytical but are ungrounded. Appending abstract nouns (paradox, trap, creep) to domain words.
Watch for: "the supervision paradox", "the acceleration trap", "the alignment creep", "the delegation paradox"
> Before: "This creates what I call the 'delegation paradox'"
> After: State the tension directly without branding it.

### Language patterns (8-16)

**8. AI vocabulary**
Words that appear far more often in post-2023 LLM text.
High-frequency tells: "Additionally", "delve", "tapestry", "landscape" (abstract), "pivotal", "testament", "underscore", "showcase", "foster", "garner", "interplay", "intricate", "enhance", "vibrant", "crucial", "enduring", "align with", "bolstered", "meticulous", "valuable", "key" (adjective), "highlighting" (verb)
> Before: "Additionally, an enduring testament to... in the culinary landscape, showcasing..."
> After: "also includes... remain common"

**9. Copula avoidance**
Replacing "is/are/has" with fancier constructions.
Watch for: "serves as", "stands as", "features", "boasts", "represents"
> Before: "Gallery 825 serves as the exhibition space"
> After: "Gallery 825 is the exhibition space"

**10. Negative parallelisms**
Three common forms: "Not just X, but Y" / "Not only... but also..." / "It's not X, it's Y". See also #15 for the triple-negation variant.
> Before: "It's not just about the music, it's about the community."
> After: "The community matters as much as the music."

**11. Rule of three**
Forcing ideas into groups of three for rhetorical effect.
> Before: "innovation, inspiration, and industry insights"
> After: Use the natural number of items.

**12. Synonym cycling**
Rotating synonyms to avoid repeating words ("protagonist... main character... central figure... hero").
> After: Repeat the clearest term. Repetition is fine.

**13. False ranges**
"From X to Y" where X and Y aren't on a meaningful scale.
> Before: "from the Big Bang to dark matter"
> After: List topics directly.

**14. Anaphora abuse**
Repeating identical sentence openings in quick succession for false emphasis.
> Before: "They assume users will pay. They assume developers will build. They assume the market will grow."
> After: Vary the structure. Combine related points.

**15. "Not X. Not Y. Just Z."**
Dramatic countdown negating multiple items before the actual point. Distinct from the two-part negative parallelism in #10.
> Before: "Not a bug. Not a feature. A fundamental design flaw."
> After: "It's a design flaw."

**16. Rhetorical Q&A**
Self-posed questions answered immediately. Asking questions nobody asked for dramatic effect.
> Before: "The result? Devastating."
> After: "The result was devastating."

### Style patterns (17-22)

**17. Em dash overuse (the hardest pattern to fix)**
LLMs use em dashes (— and --) far more than human writers do. This is one of the strongest AI tells and also the one you are most likely to leave in your own rewrites, because dashes feel natural to you. They are not natural at the rate you use them. A human writer might use one em dash per page. LLMs use multiple per paragraph.
Every time you are about to write an em dash, stop and use a comma, a period, or parentheses instead. If the sentence needs restructuring to work without the dash, restructure it.
> Before: "The tools are useful — even essential — for modern teams."
> After: "The tools are useful, even essential, for modern teams."
> Before: "It works — most of the time."
> After: "It works most of the time."
Zero em dashes in the final rewrite is the target. If you find yourself reaching for one, that is the habit talking.

**18. Boldface overuse**
Mechanical emphasis on terms.
> After: Remove boldface unless genuinely needed.

**19. Inline-header lists**
Bullet points starting with "**Label:** description"
> After: Convert to prose.

**20. Title Case headings**
Capitalizing All Main Words In Headings.
> After: Sentence case.

**21. Emojis in structure**
Decorating headings or bullets with emojis.
> After: Remove them.

**22. Curly quotes**
ChatGPT uses curly quotes instead of straight quotes.
> After: Use straight quotes.

### Communication patterns (23-29)

**23. Chatbot artifacts**
"I hope this helps!", "Let me know if...", "Certainly!", "Of course!", "Great question!"
> After: Remove entirely.

**24. Knowledge-cutoff disclaimers**
"While specific details are limited...", "as of my last training..."
> After: Find the actual information, or remove the hedge.

**25. Sycophantic tone**
"Great question! You're absolutely right!"
> After: Respond to the substance directly.

**26. "Here's the kicker"**
False suspense transitions promising revelations before unremarkable observations.
Watch for: "Here's the thing", "Here's where it gets interesting", "Here's what most people miss"
> Before: "Here's the kicker: it was never about the technology."
> After: Just make the point.

**27. "Think of it as..."**
Patronizing analogies assuming readers need metaphors. Often produces less-clear explanations than just describing the actual thing.
Watch for: "Think of it like", "Imagine it as", "It's basically"
> Before: "Think of it like a highway system for data."
> After: Explain the actual thing.

**28. "Imagine a world where..."**
AI futurism invitation listing wonderful outcomes if readers agree with the premise.
> Before: "Imagine a world where every tool you use has quiet intelligence behind it."
> After: State the specific capability being proposed.

**29. False vulnerability**
Simulated self-awareness or honesty that reads as performative. Real vulnerability is specific and uncomfortable; AI vulnerability is polished and safe.
> Before: "And yes, I'm openly in love with the platform model."
> After: State the actual opinion with real stakes.

### Filler and hedging (30-33)

**30. Filler phrases**
"In order to" -> "To". "Due to the fact that" -> "Because". "At this point in time" -> "Now". "It is important to note that" -> delete.

**31. Excessive hedging**
"could potentially possibly be argued that it might"
> After: "may"

**32. "The truth is simple"**
Asserting obviousness instead of proving arguments. Declaring simplicity rather than demonstrating it.
Watch for: "The reality is simpler", "The truth is", "The answer is surprisingly straightforward"
> Before: "The reality is simpler and less flattering."
> After: Present the reality and let the reader judge.

**33. Generic positive conclusions**
"The future looks bright. Exciting times lie ahead."
> After: State specific plans or facts.
