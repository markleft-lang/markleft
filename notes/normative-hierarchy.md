# Memo: what is normative, and what the conformance suite is for

*Non-normative. Status: decided 2026-08-06. Amends the "the suite IS the standard" formulation carried over from CommonMark in `.claude/decisions.md` §9 (Phase 1) and in `CLAUDE.md`. Definition material: this belongs in the definition's metrology framing, in condensed form.*

## The hierarchy

1. **Normative — the specification document**, formal grammar included. This is where the language is defined. A behavior exists because the document says so.

2. **Binding but subordinate — the conformance suite.** Derived from the document. It is how conformance is *checked*, and passing it is what an implementation must do; but it does not define what conformance is.

3. **Realizations — the reference parser, formatter, migrator, and any third-party implementation.** None is privileged, the reference parser included.

## Why the document, and not the suite

CommonMark made its example suite normative for a specific reason: it had no grammar. Emphasis there needs seventeen interlocking rules and a delimiter stack, which is not expressible as a context-free grammar, so the only precise statement of behavior available was a large set of worked examples. That was the right call under those constraints.

Markleft is not under those constraints, and paying the cost of formalization while declining its benefit would be a poor trade. Invariant 3 calls for an executable spec — grammar plus tests — and invariant 4 buys prefix-decidable, backtracking-free parsing. A formal grammar is a *documentary* artifact that is nonetheless unambiguous: it is not prose with interpretive slack. That is precisely what earns the document its normative status here.

The framing is borrowed from metrology as an intellectual model — it confers no authority and implies no institutional backing — and `.claude/decisions.md` §7 already used the correct term without following it through: the conformance suite is **a key comparison for parsers**. In metrology a key comparison is not a definition; it is how independent realizations are checked against each other and against the definition. Compare the SI: the metre is defined by a single sentence about the speed of light, and that definition has outlived every technology used to realize it, precisely because it was written to be independent of all of them.

## The suite is mandatory, not optional

A norm without a *mise en pratique* is all but useless. The failure mode of a documentary-only standard is this project's own founding exhibit: Gruber 2004 was a document, AsciiDoc had documents for two decades, and prose without executable checking drifts into interpretation until two implementations disagree in production and nobody can say which one is wrong.

So the suite is not a nice-to-have that trails the document. It ships with the document, it is exhaustive, and an implementation that has not passed it has no claim to conformance. Subordinate does not mean secondary.

## The precedence rule

**When the suite and the document disagree, the document governs and the test is presumed to be the bug — but the disagreement is triaged, because sometimes the test is the evidence that the document is wrong.**

Exactly one of three things is true, and the fix differs in each case:

1. **The test is wrong.** It asserts something the document does not say. Fix the test. This is the common case and needs no ceremony.

2. **The document means the right thing but says it unclearly.** The test did its job — it found an ambiguity, which under invariant 3 is a defect whether or not any implementation has tripped on it yet. Fix the prose or the grammar so the intended reading is the only reading. Behavior does not change; the text that determines it does.

3. **The document is clear, and wrong.** The design is defective. This is a specification revision: it goes through the decision record — and later the RFC process — never through a quiet edit, and never by letting the suite silently carry the corrected behavior on its own.

The point of "the document governs" is that it settles *where a change must land*, not *who was right*. Routing, not infallibility. The kilogram was redefined because realizations outran the artefact; a discrepant comparison is sometimes a finding about the definition. What the rule excludes is only this: a behavior that exists in the suite and nowhere in the document, which is how a standard quietly relocates into its tests.

## Consequences

- **Conformance examples assert against the specified JSON AST, not against HTML.** CommonMark's `spec.txt` asserts HTML output, which binds the standard to a serialization target unrelated to the language and is the main reason test suites age badly. HTML rendering becomes one downstream mapping among several — tested, documented, not normative.

- **No behavior exists only in the suite.** A test that asserts something the grammar does not determine is a specification defect, filed upstream. Reviewers of a new example should ask which clause determines it, and treat "none" as a finding rather than a formality.

- **Version the document; the suite follows it.** The suite records the spec revision it exercises, and the spec is never tagged ahead of the suite that exercises it.

- **Repository layout follows the hierarchy** — one repo per layer: `markleft-lang/markleft` is the definition (CC BY 4.0), `markleft-lang/tests` is the key comparison (CC BY 4.0), and `markleft-lang/markleft-rs` will hold the Rust realization (MIT, Phase 2). The namesake repo is the standard itself, which states the hierarchy in the org listing before anyone reads a word of it. See `notes/repo-layout.md`.

## What changed, and why it is recorded here

The earlier formulation — "the test suite is the standard" — was inherited from CommonMark along with its spec-as-tests methodology, which is genuinely one of this project's best inheritances. Only the normative claim is amended, not the methodology. The suite remains exhaustive, executable, and mandatory. What moves is the definition, which now lives where a standard that intends to outlast its implementations should keep it.
