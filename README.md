# Markleft

> **Everything that is not prose leaves a mark.**

A formally specified, ambiguity-free successor to Markdown: the same size and the same prose-like source, with a formal grammar, an executable conformance suite, and exactly one parse for every input.

**This repository is the standard.** Not a parser that happens to define the language by its behavior — the specification itself. Implementations live elsewhere, and none of them is privileged.

## One rule for every inline construct

**A sigil means nothing unless the very next character is `{` — or `[`, when the construct carries its own text.**

| Construct | Mark |
| --- | --- |
| Emphasis | `*{…}` |
| Strong | `**{…}` |
| Emphasis and strong | `***{…}` |
| Superscript | `^{…}` |
| Subscript | `_{…}` |
| Anchor | `#{…}` |
| Link | `@{…}` |
| Link carrying its own text | `@[text]{…}` |
| Image | `!{…}` |
| Image carrying alt text | `![alt]{…}` |
| Escape | `\{…}` |
| Verbatim | `` `…` `` |
| Verbatim with a decorator | `` `…`{label} `` |

So the whole language is three sentences. **Blocks** are decided by position in the line — the start opens one, the end breaks one. **Inlines** are decided by a sigil immediately followed by a brace. **Verbatim** is delimited by matching backtick runs, because content that is not interpreted cannot be closed by balancing.

Everything else follows, including the part that matters most: `5 * 3 = 15`, `snake_case`, `2^32`, `[sic]`, `C:\Users\name`, `#hashtag`, `@handle`, and `{"key": 1}` are all ordinary text, and none of them needs an escape.

## Six guarantees

- **Prose-safety.** Natural-language prose renders verbatim. Money, snake_case, `5 * 3 = 15`, and shell snippets are safe by construction, never by heuristic — there is no flanking rule and no lookahead anywhere in the language.

- **Verbatim means verbatim.** Content inside a fence or a span is carried through untouched; TeX backslashes survive. Where a delimiter appears in your content, you lengthen the delimiter — one escape mechanism, at every size, with no exception list.

- **Five minutes.** The complete core fits on one reference card. No rule has an "unless" clause.

- **One meaning.** Every input has exactly one parse, and the specification is executable — a grammar and a conformance suite, not interpretable prose.

- **Linear time.** A single pass, prefix-decidable blocks, no backtracking, no pathological inputs.

- **Read it anywhere.** You can always read the entire document in plain text, with no compiler, viewer, or extension. Nothing renders that the source does not already say.

## What Markleft is not

**Not Markdown, and not compatible with it.** Markleft keeps Markdown's block layer — headings, lists, block quotes, fences, tables — and replaces its inline layer entirely. A Markdown document does not parse as intended here, and a Markleft document does not render as intended there. That break is deliberate: near-compatibility is exactly what produced Markdown's flavour drift, where every implementation is *almost* the same as every other and a writer cannot tell whether their document survives a move between platforms. A platform can support both languages side by side, as two languages with two file extensions. What stops paying is the temptation to stretch one implementation across both.

The spirit is inherited; the syntax is not. What Markleft takes from Markdown is the thing Markdown got right — that a source file should read as prose, and that the marks should be few and quiet.

Not a feature platform, either. Markdown's problem was never a shortage of features — it was that the same source produces different documents. The job here is to remove that ambiguity while keeping source that reads as plain text and a core small enough to learn in one sitting.

It is a little richer than Markdown where the richness pays for itself. Math is part of the core in the way that matters: `$` is never math syntax, and there is a first-class construct — fenced and inline — whose content is verbatim and safe from reinterpretation. A renderer still chooses the typesetting, as it always did; what it can no longer do is change what the source says. Decorators and pipe tables earn their place too, several of them adopted from djot, whose design solved these problems well and is credited throughout. Every addition clears the same bar: it removes an ambiguity or fills a genuine gap, **and it costs nothing in plain-text clarity**. A construct that makes ordinary, unmarked prose read worse does not clear the bar, however much it offers elsewhere.

**The five-minute property is the budget.** Most of the language decisions remove something; a few add; the total still has to fit on one reference card. "Markleft could also do X" is therefore not an argument on its own — the argument has to be that X is worth what it costs every reader who never uses it.

Concretely, Markleft is not a document-preparation system (Typst, Quarkdown, Quarto), not an extension layer grown over an ambiguous base (MyST, MDX, Markdoc), and not a dialect, superset, or flavour of Markdown.

**The name turns out to describe the design.** Every construct's mark sits to the left of its content: blocks mark the left of a line, inlines mark the left of a span. That was not planned — the name came first — and it is where the tagline at the top comes from. Read it backwards and it is the guarantee: no mark, therefore prose.

## What is normative

Everything in this repository is part of the standard, **except `CLAUDE.md` and `.claude/`**, which are working scaffolding — a decision record, a prior-art survey, and editing conventions. They are neither normative nor part of what gets released. `LICENSE` is the Creative Commons licence text, reproduced verbatim.

Within the standard, `notes/` holds non-normative memos: design challenges, and the reasoning behind decisions the normative text does not explain about itself.

The specification document governs. The conformance suite in [markleft-lang/tests](https://github.com/markleft-lang/tests) is how conformance is *checked* — mandatory for any conformance claim, and binding, but subordinate to this document. Where the two disagree, this document governs and the test is presumed to be at fault — unless the test turns out to be the evidence that the specification needs revising. "Governs" settles where a change lands, not who was right.

## Status

**Version 0.1.0.** Phase 0 — the definition is drafted and no implementation has shipped. The version bands are stated in the definition, section 12: `0.x.0` when the language changes, `0.x.y` for everything else, `1.0.0` when this document is normative and a realization passes the conformance suite.

The `.md` files in this repository are Markdown, and they are not written in Markleft. They cannot be: a document containing a link is one language or the other, and GitHub renders only the first. Markleft sources arrive when the reference parser and the canonical formatter do.

The phase checklist on the [organization profile](https://github.com/markleft-lang) is the status board, and it is the single source of truth. It is kept there rather than here because the work spans repositories: the conformance suite lands alongside the grammar, and the reference parser arrives in a repository that does not exist yet.

## Copyright

Crown copyright, Canada — 2026. An author is a Canadian federal public servant, so the Copyright Act vests copyright in the Crown by operation of law. **This is not sponsorship.** No department, agency, or institution owns, funds, backs, endorses, reviews, or governs Markleft. It is an independent project, maintained by its authors.

## Licensing

**[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).** Copy it, quote it, adapt it, implement it, and sell what you build on it. Attribution is the only condition.
