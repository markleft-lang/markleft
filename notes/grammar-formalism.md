# Memo: the grammar formalism, surveyed and settled

*Non-normative. Status: survey recorded 2026-08-10. The Phase 1 choice in `.claude/decisions.md` §9 — PEG for inlines, a small-step block algorithm, spec-as-tests — was checked against the 2026 state of the art and stands confirmed. Three execution rules trickle out of the survey and are recorded here; none changes the language, all change how the grammar lands. When Phase 1 opens, anything here that binds an artifact goes through the decision record as an amendment, per the working convention.*

## The question

Is BNF still the instrument for a formal language specification, and if not, what is? The question matters because invariant 3 says the spec is executable, not interpretable prose — so the formalism is not a typesetting choice, it is the mechanism that makes the one-meaning property a checkable claim instead of a promise.

## The survey, in brief

**BNF survives as a family of dialects, all of them recognition-only.** ABNF (RFC 5234) still carries every IETF protocol; the W3C's EBNF still carries the XML spec. Both say what matches without saying what the parse tree is, neither can express "first column of the line" without a prose escape hatch, and — the decisive count — both are context-free-grammar notations, and a CFG can be *ambiguous*. A CFG grammar would leave us proving the one-parse property separately, which surrenders half of invariant 3 to hand-written argument.

**PEG (parsing expression grammars, Ford 2004) is the Markleft-shaped formalism.** Ordered choice makes ambiguity impossible by construction — a PEG denotes a recognizer, not a language, so "exactly one parse" is a theorem that comes free rather than a property to defend. Packrat parsing gives guaranteed linear time. Invariants 3 and 4, by construction, from the notation itself. The known weakness is the PEG equivalent of ambiguity — an ordering mistake between overlapping alternatives — and the conformance suite is the guard against it. This is why §9 chose PEG, and the survey found nothing that has displaced it.

**Line-oriented block structure does not fit grammar notation, and the honest specs say so.** HTML5 specifies its parser as a state machine in numbered prose steps; CommonMark specifies its block strategy as a phased algorithm in an appendix. CommonMark famously has no grammar at all — not from laziness, but because Markdown as designed cannot have one. Markleft's prefix-decidable blocks were designed precisely so that the block algorithm can be short, total, and normative. A "grammar" for Markleft is therefore honestly two artifacts — a block algorithm and an inline PEG — and saying so plainly is more rigorous than stretching one EBNF over both.

**The current gold standard is WebAssembly's shape.** Every Wasm feature is specified three ways at once — formal semantics, prose, and an executable reference interpreter — kept honest by one official test suite; in 2025 the group adopted SpecTec, a toolchain that generates the typeset spec and an executable interpreter from a single source, and the generated interpreter found real bugs in the hand-written spec. Full SpecTec is more machinery than a language this size needs. The *shape* — definition, grammar, reference realization, suite, cross-checked so a wrong claim is a failing test rather than a debate — is the metrology framing this project already carries, arrived at independently. Worth naming in `.claude/landscape.md`'s attribution list when the prior-art section is written.

**Tree-sitter is worth shipping and can never be normative.** It is the grammar format editors actually consume — GitHub highlighting, Neovim, Zed — so a tree-sitter grammar is most of Phase 4's editor-highlighting deliverable. It is also GLR-based: it *tolerates* ambiguity and resolves it with precedence annotations, so it cannot carry the one-meaning property. That combination decides its status exactly.

**Executable-EBNF toolchains (ANTLR, Lark, Instaparse) offer the wrong guarantees.** Fine tools; none gives linear time *and* unambiguity by construction, so each would reintroduce the proof obligations PEG discharges for free. Set aside on that count alone.

**On "LLM-proof":** notation choice barely matters to a language model — ABNF, PEG, and prose algorithms are all equally legible. What makes a spec proof against confident misreading, human or machine, is *checkability*: a claim about the language should be falsifiable by running the suite. The ranking is spec-as-tests, then executable grammar, then formal-but-inert notation, then prose. Markleft holds one unusual card here: the whole grammar fits in a model's context window, which is what makes Phase 4's system-prompt-sized description possible at all.

## The three rules that trickle out

**1. The grammar is a file the parser cannot drift from.** The inline PEG lands as a machine-readable file under `grammar/`, and the reference parser is generated from it or tested against it in CI — never hand-transcribed into `markleft-rs`. The moment the grammar file and the parser can drift apart, the spec is interpretable prose again, which is the CommonMark situation the whole cost of formalization is paid to escape.

**2. The block algorithm is numbered small steps, each exercised by named conformance examples.** §9 already asks the algorithm to be honest about the non-CFG bits (fence-length matching); this sharpens the deliverable: every numbered step points at the examples in `tests` that exercise it, so the algorithm is checkable step by step rather than only end to end.

**3. A grammar in a notation that tolerates ambiguity is a realization, never the definition.** Tree-sitter concretely: ship it at Phase 4, run the suite against it like any realization, let it govern nothing. The same sentence pre-answers any future "why not just use the ANTLR grammar as the spec" — a notation that needs precedence annotations to pick a parse has already given up the property the definition exists to guarantee.

A corollary rather than a fourth rule: the Phase 4 system-prompt-sized description is validated by running its output through the suite, not by proofreading it. It is a realization of the spec in prose, and realizations are privileged over nothing.

## What this memo does not do

It does not reopen §9, which chose the formalism; it confirms it against the field and records why the alternatives stay set aside. It adds no syntax, so `.claude/grammar.md` carries nothing from it. And it does not commit to any PEG toolchain or file format — that is a Phase 1 call, to be made when the grammar is written and recorded then.
