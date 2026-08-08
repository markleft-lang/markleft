# Markleft

A formally specified, ambiguity-free successor to Markdown: the same size and the same prose-like source, with a formal grammar, an executable conformance suite, and exactly one parse for every input.

**This repository is the standard.** Not a parser that happens to define the language by its behavior — the specification itself. Implementations live elsewhere, and none of them is privileged.

## Six guarantees

- **Prose-safety.** Natural-language prose renders verbatim. Money, snake_case, `5 * 3 = 15`, and shell snippets are safe by construction, never by heuristic.

- **Verbatim means verbatim.** A backslash before any character yields that character — line endings included, no exception list. Any means any: the full Unicode range is text.

- **Five minutes.** The complete core fits on one reference card. No rule has an "unless" clause.

- **One meaning.** Every input has exactly one parse, and the specification is executable — a grammar and a conformance suite, not interpretable prose.

- **Linear time.** A single pass, prefix-decidable blocks, no backtracking, no pathological inputs.

- **Read it anywhere.** You can always read the entire document in plain text, with no compiler, viewer, or extension. Nothing renders that the source does not already say.

## What Markleft is not

Not a feature platform. Markdown's problem was never a shortage of features — it was that the same source produces different documents. The job here is to remove that ambiguity while keeping what made Markdown win: source that reads as plain text, a core small enough to learn in one sitting, and familiarity for anyone who already writes Markdown.

It is a little richer than Markdown where the richness pays for itself. Math is part of the core in the way that matters: `$` is never math syntax, and there is a first-class construct — fenced and inline — whose content is verbatim and safe from reinterpretation. A renderer still chooses the typesetting, as it always did; what it can no longer do is change what the source says. Decorators and pipe tables earn their place too, several of them adopted from djot, whose design solved these problems well and is credited throughout. Every addition clears the same bar: it removes an ambiguity or fills a genuine gap, **and it costs nothing in plain-text clarity**. A construct that makes ordinary, unmarked prose read worse is declined regardless of merit.

**The five-minute property is the budget.** Most of the language decisions remove something; a few add; the total still has to fit on one reference card. "Markleft could also do X" is therefore not an argument on its own — the argument has to be that X is worth what it costs every reader who never uses it.

Concretely, Markleft is not a document-preparation system (Typst, Quarkdown, Quarto), not an extension layer grown over an ambiguous base (MyST, MDX, Markdoc), and not a superset of Markdown.

## What is normative

Everything in this repository is part of the standard, **except `CLAUDE.md` and `.claude/`**, which are working scaffolding — a decision record, a prior-art survey, and editing conventions. They are neither normative nor part of what gets released. `LICENSE` is the Creative Commons licence text, reproduced verbatim.

Within the standard, `notes/` holds non-normative memos: design challenges, and the reasoning behind decisions the normative text does not explain about itself.

The specification document governs. The conformance suite in [markleft-lang/tests](https://github.com/markleft-lang/tests) is how conformance is *checked* — mandatory for any conformance claim, and binding, but subordinate to this document. Where the two disagree, this document governs and the test is presumed to be at fault, unless the test turns out to be the evidence that the specification needs revising.

## Status

Phase 0. The definition is being drafted; no implementation has shipped.

The phase checklist on the [organization profile](https://github.com/markleft-lang) is the status board, and it is the single source of truth. It is kept there rather than here because the work spans repositories: the conformance suite lands alongside the grammar, and the reference parser arrives in a repository that does not exist yet.

## Copyright

Crown copyright, Canada — 2026. An author is a Canadian federal public servant, so the Copyright Act vests copyright in the Crown by operation of law. **This is not sponsorship.** No department, agency, or institution owns, funds, backs, endorses, reviews, or governs Markleft. It is an independent project, maintained by its authors.

## Licensing

**[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).** Copy it, quote it, adapt it, implement it, and sell what you build on it. Attribution is the only condition.
