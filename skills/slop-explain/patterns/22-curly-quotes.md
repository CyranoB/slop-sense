# Pattern 22: Curly quotes

**Typographic ("smart") quotes appearing in contexts where straight quotes would be standard.**

## Why LLMs do this

ChatGPT in particular converts straight quotes to curly quotes by default in its output. The behavior comes from training on typeset prose (books, magazines, well-edited articles) where curly quotes are correct, combined with rendering settings that default to typography-friendly characters.

Other LLMs vary, but the pattern is associated strongly enough with ChatGPT that the curly quote alone is treated as a likely AI tell.

## Why it reads as AI

In informal web writing, technical documentation, code, and most blogging, straight quotes are the norm. Most keyboards produce straight quotes by default. When a user pastes prose into a chat box or types directly, they get straight quotes. So when curly quotes appear in a casual context, the reader's brain notes the mismatch.

The mismatch is especially visible in code blocks or technical content, where curly quotes are actively wrong. They will break a string literal or fail a JSON parser.

The pattern is one of the most reliable "ChatGPT did this" tells, more reliable than any vocabulary pattern, because it is unconscious. Most users do not even know how to type a curly quote on purpose.

## Examples

After (straight quotes, the ASCII " character):
> She said "the meeting was a disaster" and walked out.

Before (curly quotes, the typographic characters U+201C and U+201D):
> She said “the meeting was a disaster” and walked out.

The shift is subtle visually but obvious to anyone running a script over the text, and to many readers' eyes once they know to look. The left and right typographic quotes are different Unicode characters from the straight double-quote.

## How to self-spot

Search your draft for the Unicode characters. The four common offenders are U+2018, U+2019, U+201C, and U+201D (left and right single and double typographic quotes). Replace them with straight quotes (' and ") unless the publication or rendering context specifically calls for typographic quotes.

If you write in a tool that auto-converts (Word, Google Docs, some Markdown editors), turn off the auto-conversion or run a normalization pass before publishing.

A regex like `[‘’“”]` will find all of them. A quick `tr` or sed substitution will fix them.

## Related patterns

- **Pattern 17 (em dash overuse):** also a typographic-character pattern. ChatGPT outputs both.
- **Pattern 18 (boldface overuse):** another formatting tell from the same generation pipeline.
