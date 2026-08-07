# Markleft

A formally specified, ambiguity-free successor to Markdown: the same size and the same prose-like source, with a formal grammar, an executable conformance suite, and exactly one parse for every input.

**This repository is the standard.** Not a parser that happens to define the language by its behavior — the specification itself. Implementations live elsewhere, and none of them is privileged.

## Four guarantees

- **Prose-safety.** Natural-language prose renders verbatim. Money, snake_case, `5 * 3 = 15`, and shell snippets are safe by construction, never by heuristic.

- **Five minutes.** The complete core fits on one reference card. No rule has an "unless" clause.

- **One meaning.** Every input has exactly one parse, and the specification is executable — a grammar and a conformance suite, not interpretable prose.

- **Linear time.** A single pass, prefix-decidable blocks, no backtracking.

## What is normative

Everything in this repository is part of the standard, **except `CLAUDE.md` and `.claude/`**, which are working scaffolding — a decision record, a prior-art survey, and editing conventions. They are neither normative nor part of what gets released.

Within the standard, `notes/` holds non-normative memos: design challenges, and the reasoning behind decisions the normative text does not explain about itself.

The specification document governs. The conformance suite in [markleft-lang/tests](https://github.com/markleft-lang/tests) is how conformance is *checked* — mandatory for any conformance claim, and binding, but subordinate to this document. Where the two disagree, this document governs and the test is presumed to be at fault, unless the test turns out to be the evidence that the specification needs revising.

## Status

Phase 0. The charter is being drafted; no implementation has shipped.

## Licensing

CC0 intent. The instrument is not yet settled, because Crown copyright interacts non-trivially with CC0 as a waiver, so this material currently carries "Crown copyright — to be released CC0 pending review".

Crown copyright applies because an author is a Canadian federal public servant, by operation of law. Markleft is an independent project: no institution sponsors, endorses, reviews, or governs it.
