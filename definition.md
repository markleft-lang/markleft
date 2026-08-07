# Markleft — The Definition

*Crown copyright, Canada — 2026. Licensed CC BY 4.0.*

**Why "Definition".** The word is the one place computing and metrology converge. Metrology *defines* the metre in a sentence and checks every realization against that definition; computing's most rigorous language document is *The Definition of Standard ML*. It also names precisely what this document is **not** — a realization. This file was called `charter.md` until 2026-08-07; in IETF and W3C practice a charter is a working group's governance document and never the technical specification, so the name was signalling the wrong thing. If the licensing, governance, and versioning sections ever split off, *that* document is legitimately the charter.

**Status: scaffold.** Section headings and scope notes only. **No text in this file is normative yet.** Each section names its source material so drafting never starts from a blank page; sources under `.claude/` are working scaffolding and are not themselves part of the standard.

Voice: public-standard sober. A standard has to read like one. The humor budget for this document is the "what's left" reading in the Name section and nothing else.

## 1. Purpose and scope

What Markleft is, in one page: Markdown's size and prose-like source, with a formal grammar, an executable conformance suite, and exactly one parse for every input. State the problem it exists to fix — the same source producing different documents — and the founding exhibit from `.claude/decisions.md` §1.

## 2. Non-goals

Two distinct claims, both load-bearing, from `CLAUDE.md`:

- Not a toolchain and not a document-preparation system. No templating, executable code cells, scripting, page awareness, or output-format directives.

- Nothing renders that is not in the source (decision 15). The cardinal rule this section exists to protect: a reader can always read the entire document in plain text, with no compiler, viewer, or extension. State it here as a promise to the reader, and let section 4 carry the rule.

- Not a richer Markdown for its own sake — and not an austerity project either. Additions must remove an ambiguity or fill a genuine gap *and* cost nothing in plain-text clarity. The five-minute property is the budget.

## 3. The five invariants

Prose-safety, the five-minute property, the one-meaning property, the linear-time property, the cardinal rule. Constitutional: a feature violating one is rejected regardless of merit. Source: `.claude/decisions.md` §3.

State plainly that there are five invariants and that the READMEs list six *guarantees* — user-facing promises derived from the invariants and decisions. The counts differ on purpose, and this section is where the relationship is explained rather than left to inference.

The cardinal rule needs its enabling framing stated here, not just its prohibition: it is the minimal set that lets capability live in tooling and be implemented cleanly. Features are relocated, not forbidden — see the non-goals in section 2 and `.claude/decisions.md` §2.

## 4. The binding decisions

Each decision gets its own subsection: the rule, the rationale, and the alternatives rejected. Source: `.claude/decisions.md` §4, whose numbering is normative — cite decisions by number here and everywhere else.

Decisions 4, 5, and 6 (ATX-only headings, fenced-only code, strict lists) carry the highest muscle-memory risk and deserve the fullest treatment. Decision 14 carries open riders that this document must settle rather than inherit: byte-order mark handling, invalid UTF-8, anchor and label matching, and the definition of "column" outside ASCII.

## 5. Prior art and inheritances

What this project takes, and from whom, with attribution rather than around it. Source: `.claude/landscape.md`, including the positioning table and the explicit inheritance list — CommonMark's spec-as-tests methodology and byte-compatibility baseline, djot's linear-time architecture and uniform escaping, MyST and reStructuredText's directive concept, Markdoc's validation demand, Typst's error-message bar, the Eclipse AsciiDoc TCK concept.

Most of the syntax here is djot-vetted. Saying so is both accurate and disarming.

## 6. The name

Four readings, all load-bearing; the copyleft lineage; the two morphological axes and why Markleft sits on the succession axis rather than the flavor axis; the availability accounting. Source: `.claude/decisions.md` §5 and §8.

## 7. Metrology framing

One definition, many independent realizations, kept honest by comparison between them. The specification is the definition; the conformance suite is the key comparison; the reference parser is one realization and not the privileged one. Source: `notes/normative-hierarchy.md`.

The framing is borrowed as an intellectual model. It confers no authority and implies no institutional backing — see section 9.

## 8. Conformance

What it means to be Markleft-conformant, and how a claim is checked. The normative hierarchy and its precedence rule belong here in condensed form: when the suite and this document disagree, this document governs and the test is presumed to be the bug — unless the test is the evidence that this document needs revising, which is a specification revision and never a quiet edit.

Reserve the possibility of a lightly-governed "Markleft-conformant" mark, held by the project's maintainers. Do not promise the mark; reserve it.

## 9. Copyright, licensing, and non-endorsement

Crown copyright applies by operation of law, because an author is a Canadian federal public servant. It is not sponsorship, endorsement, review, or stewardship, and this document must say so in terms a reader cannot misread.

CC0 intent for this specification and for the conformance suite; MIT for code. The instrument is unsettled while Crown copyright's interaction with CC0-as-waiver is resolved. Exact notice wording is for counsel — every notice string in these repositories is a placeholder.

## 10. Governance

Markleft is maintained by its authors and governed by no institution. Say what the process is today — one maintainer decides and records the decision — rather than describing a committee that does not exist. Name the trigger for something more: more than one decision-maker, at which point the decision record becomes an RFC process.

## 11. Versioning and change process

How the specification is versioned, how the conformance suite records the revision it targets, and why the specification is never tagged ahead of the suite that exercises it. Source: `notes/repo-layout.md`.

## Appendix A — Deltas from Markdown

Normative list of every divergence from CommonMark, doubling as the migration guide. Maintained in `deltas.md` and incorporated here by reference.
