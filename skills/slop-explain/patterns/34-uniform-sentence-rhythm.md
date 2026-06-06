# Pattern 34: Uniform sentence rhythm (low burstiness)

**Sentences that barely vary in length or shape, producing a flat, even cadence with no rhythmic surprise.**

## Why LLMs do this

A language model generates each sentence by predicting the most probable continuation. Left to its own devices, that process converges on a comfortable middle: clauses of similar weight, joined in similar ways, landing at a similar length. There is no internal pressure toward variety, because variety is not what "most probable next token" optimizes for. The safe, well-formed, medium-long sentence is always available, so the model reaches for it again and again.

Human writers vary sentence length for reasons a model does not feel: to control pace, to land a point, to breathe. We write a long sentence that builds and qualifies and accumulates, and then we stop short. The short one hits harder because the long one set it up. That contrast — researchers call it "burstiness" — is a fingerprint of a mind deciding, sentence by sentence, how much room each thought needs. The model is not deciding that. It is averaging.

## Why it reads as AI

Burstiness is one of the two signals (with perplexity) that statistical AI detectors like GPTZero weigh most heavily, precisely because it is so hard to fake by accident and so consistently absent from raw model output. A passage where every sentence runs 18 to 24 words, each a tidy subject-verb-object with one subordinate clause, reads as machine-smooth even when the vocabulary is clean and the argument is sharp. This is what reviewers are reacting to when they call writing "robotic", "formulaic", or "lacking creative grammar" without being able to point at a single bad word.

The tell is structural, not lexical, which is why a text can score "very human" on a word-level slop checker and still get flagged 90%+ by a perplexity model. The words are fine. The rhythm is a metronome.

## Examples

Before:
> The invention happened in Toronto, and the company that captured what it was worth happened somewhere else, and that gap between where the breakthrough was made and where the money landed is the whole story.

After:
> The invention happened in Toronto. The company that captured what it was worth happened somewhere else. That gap is what this whole piece is about.

Before:
> The framework is fast and well-documented, and it has a large community of contributors, and the maintainers respond quickly to issues, which makes it a reliable choice for production work.

After:
> The framework is fast and well-documented. It has a real community. The maintainers actually answer issues. For production work, that reliability is the whole game.

Notice that the human versions swing: a long sentence next to a three-word one, a fragment, a clause that runs on by design. The AI versions hold one length and one shape.

## How to self-spot

Read your draft and mark the word count of each sentence. If the numbers cluster in a narrow band — say, almost everything between 15 and 25 words — you have a burstiness problem even if nothing else is wrong.

Look for runs of three or more consecutive sentences with the same structure (subject, verb, object, trailing clause). Break one. Add a short declarative. Let a fragment stand. Combine two short ones into a long one so a later short one can punch.

The `rhythm.py` script in the `slop-sense` skill reports this directly as a coefficient of variation (CV): below about 0.40 is the uniform cadence detectors flag; healthy human prose tends to run higher.

## Related patterns

- **Pattern 35 (aphoristic paragraph closers):** the rhythmic counterpart at the paragraph level — same length, same shape, ending after ending.
- **Pattern 36 (reflexive formality):** another structural tell invisible to word-level scorers.
- **Pattern 14 (anaphora abuse):** the opposite failure — repetition so identical it draws attention — but both come from the model averaging instead of deciding.
