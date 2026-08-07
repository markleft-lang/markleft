# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

**Markleft** is a formally specified, ambiguity-free successor to Markdown: Markdown's size and prose-like source, with a formal grammar, an executable conformance suite, and exactly one parse for every input. **Independent project, maintained by its authors — no institutional steward, sponsor, or endorser.** Crown copyright attaches by operation of law, which is not sponsorship; see Licensing below, and never imply otherwise anywhere. GitHub org: `markleft-lang`. See `.claude/decisions.md` for the full decision record and rationale — read it before proposing design changes. See `.claude/plan.md` for the running plan — current work, parked ideas, errands, and open questions in plain language; add to it as things come up. See `.claude/landscape.md` for the prior-art survey (djot, CommonMark, MyST, Quarto, Markdoc, Typst, AsciiDoc, etc.) — consult it when writing the definition's prior-art section or evaluating features other languages have.

The name: markup went up, markdown went down, Markleft moved sideways — and the language is *what's left* after fifteen years of curation. When someone asks "what's left?", that question is the elevator pitch.

## Non-negotiable invariants

Every contribution must preserve all five. A feature that violates one is rejected regardless of merit.

1. **Prose-safety:** natural-language prose renders verbatim; money, snake_case, `5 * 3 = 15`, shell snippets are safe *by construction*, never by heuristic.

2. **Five-minute property:** the complete core fits on one reference card; no rule has an "unless" clause.

3. **One-meaning property:** every input has exactly one parse; the spec is executable (grammar + tests), not interpretable prose.

4. **Linear-time property:** single pass, prefix-decidable blocks, no backtracking.

5. **Cardinal rule:** you can always read the entire document in plain text, with no compiler, viewer, or extension; nothing renders that the source does not already say. **Enabling, not restrictive** — the minimal set that lets capability live in tooling and be built cleanly. Detail: decision 15.

**Non-goals (definition-level, from `.claude/landscape.md`):** Markleft is a language, not a toolchain and not a document-preparation system. No templating, no executable code cells, no scripting, no page/layout awareness, no output-format directives. Those make a different product (Quarto, Typst, Quarkdown) and every one of them front-loads complexity that kills the five-minute property. Feature requests in that direction are out of scope, not merely deferred. Decision 15 turns this from a scoping statement into a syntax rule: **nothing renders that is not in the source**, so no TOC directive, no auto-numbering, no transclusion — you can always read the whole document in plain text, with no compiler or viewer.

**Not a richer Markdown for its own sake — but not an austerity project either.** Markleft *does* add: math in the core, decorators, pipe tables, several adopted from djot. The rule is that every addition pays for itself. It must remove an ambiguity or fill a genuine gap, **and cost nothing in plain-text clarity** — a construct that makes ordinary unmarked prose read worse is declined regardless of merit, because prose-safety is invariant 1 and richness is not an invariant at all.

**The five-minute property is the budget.** Most of the binding decisions remove something; a few add; the total still fits on one reference card. So "Markleft could also do X" is never an argument by itself — the argument must be that X is worth what it costs every reader who never uses it. When weighing a djot feature we have not adopted (smart punctuation, definition lists, generic containers), that is the question to ask, and `.claude/landscape.md` records why djot is the right place to shop.

## Core language decisions

Summary only — rationale and rejected alternatives live in `.claude/decisions.md` §4. Numbering is normative and matches that section: cite decisions by number ("challenge to decision 6"), here, in commits, and in `notes/` memos.

1. Bare `$` is ALWAYS literal text. Math is core: `` `x=y`{math} `` inline, ```` ```math ```` block. Math content is verbatim — TeX backslashes survive. "Core" = three guarantees: `$` is never syntax, the construct exists at both sizes with verbatim content, the label reaches the AST intact. `math` is a conventional label, not a reserved word (decision 9).

2. Emphasis: `*em*` / `**strong**` only. Underscore is not syntax. `{*...*}` for intra-word.

3. Backslash before ANY character escapes it. No exception list — explicitly including a line ending, which is the visible line break.

4. ATX headings only, one closed form: one to six `#` in the first column, a space, then non-empty text; any line that doesn't match is text. No setext, no closing `#` run, no indentation, no empty heading.

5. Fenced code only. No indented code blocks.

6. Strict list rules: one marker per list, content-column alignment, no lazy continuation. *Highest muscle-memory risk, with 4 and 5.* *Open rider: whether ordered-list numbers render as written — CommonMark renumbers from the first marker, which manufactures content decision 15 forbids.*

7. No raw passthrough at all — no raw block, no raw span, no `=format` decorator. A Markleft document is inert by construction. Raw HTML would be a decision 15 hole (`<iframe>` is transclusion).

8. GFM-style pipe tables, strictly gramma­rized.

9. Decorators — one vocabulary, two positions: ```` ```math ````, ```` ```math #eq1 ````, ```` ```#snippet-3 ```` on fences; `` `x`{math #eq1} `` inline. Two token shapes, **each at most one, no multiplicity exception anywhere**: bare word (format label, kebab-case ok) and `#id` (an anchor — navigation, not style; duplicate ids across a document are a lint check, not a parse error). **There is no `.class`** — it was the only construct whose purpose was to let rendering vary, it strips to nothing on platforms that sanitize CSS, it balloons the source, and it invites mixing styling with content. No `key=val`, no `=format`. **No word is ever reserved:** labels are opaque, carried into the AST, and no decorator word can affect the parse. Meanings (`math`, `rust`) are convention, held in a non-normative appendix and the validator's list; unknown words parse as plain verbatim and warn on lint. **Bare ```` ``` ```` and `` `…` `` mean verbatim plain text, period** — that is a definition, not a default. No `text` label: not forbidden (nothing is reserved), but never blessed — the validator calls it redundant and the formatter strips it.

10. Tabs are not structural (validator warning + auto-fix).

11. Deltas from djot: no smart punctuation in core; keep Markdown muscle memory where Markdown wasn't broken (links, blockquotes, backtick verbatim — challenged on keyboard-accessibility grounds and kept, `notes/backtick-verbatim-challenge.md`).

12. Match CommonMark byte-for-byte wherever it's unambiguous and harmless; every divergence goes in the "Deltas from Markdown" appendix, which doubles as the migration guide.

13. Hard breaks are explicit: `\` before a line ending is the only one. Trailing-space breaks are removed, and trailing whitespace is never significant.

14. Unicode is the character set, UTF-8 the encoding; every code point is text and content is never normalized. *Open riders for the definition: BOM, invalid UTF-8, anchor/label matching, what "column" means for decision 6 outside ASCII, and what character set a decorator word, class, and id may use (decision 9).*

15. No generated content — the source is the whole document. No TOC directive, auto-numbering, index, transclusion, or variable substitution. **Cardinal rule: you can always read the entire document in plain text, with no compiler, viewer, or extension.** The source need not *look* like the rendered form (`**bold**` is not bold); it must be *semantically equivalent* to it, up to format. Test for any construct: is it changing the presentation of written content, or manufacturing content? Presentation instructions stay open; manufacturing is closed. **Back half:** rendering is lossy on marks and lossless on content — a decorator disappears into the typography that re-expresses it, so every format word must name something a renderer visibly distinguishes.

17. Anchors are positional: `{#id}` valid **anywhere**, marks a point, not an attachment — so no paragraph exclusion and no adjacency rule. **An anchor marks where the reader arrives** (lead for anything longer than a line, trail on a heading). Whitespace around it is optional and insignificant. **References need no new syntax:** `[text](#id)`, `[text](file.md#id)`, and `[text](https://…#id)` are one construct — a link to a URL fragment. Markleft's xref contribution is *removing the slug heuristic*, not adding a feature. No bare sigil (`#id`/`@id`/`@#id`): it would have to generate its own text, and bare `#id` is prose-unsafe. Validator stays single-document; a target with a path or host is a URL and is not validated.

16. Images are core, `![alt](src)` unchanged from CommonMark — no delta. Legal because **an image adds no text**: decision 15 governs the document's text, so "no transclusion" there means no *textual* transclusion. Alt is optional; empty alt is a lint warning, never an error. **The memorable rule: `[…]` links to the target, `![…]` shows it instead — the `!` is a presentation switch, not a different construct.**

## File extensions

Both forms are DECIDED (`.claude/decisions.md` §6). Canonical pair, exactly two: `.markleft` (long form — documentation-canonical, the collision-proof long-term anchor) and `.lf` (short form — the `.md` two-letter echo and the line-feed wink). Never use: `.lft` (runner-up, reserve fallback only — not a live option), `.left`, `.mklf` (retired), `.ml`, `.mll` (OCaml/ocamllex), `.mlf`, `.mlt`, `.mlk`, `.mkt`, `.mf`.

## Licensing & copyright (apply to every new file)

**Crown copyright applies by operation of law, not by choice and not as sponsorship.** The author is a Canadian federal public servant, so Copyright Act s.12 vests copyright in the Crown whatever anyone would prefer. It carries no endorsement, no review authority, and no governance role over this project. Never phrase a notice so that a reader could infer a department backs, owns, supervises, or vouches for the language.

- **Spec & conformance suite: CC BY 4.0** *(settled 2026-08-07)* — the most open CC licence short of CC0: attribution only, no share-alike, no non-commercial, no no-derivatives. Chosen because OGL-Canada 2.0, the federal open-licensing instrument, grants substantively the same rights. Mark files "© His Majesty the King in right of Canada, 2026. Licensed CC BY 4.0." **Public files state the position; they never show the reasoning or hedge it** — the wording is still provisional pending counsel, but that fact lives here and in the decision record, not in a notice. CC0 remains the stated *intent*, blocked on authority rather than law — a public servant cannot dedicate Crown material on the Crown's behalf. **Do not claim OGL-Canada declares CC BY compatibility; it does not** — that clause is the UK OGL's.

- All code: MIT.

- Exact notice wording is a question for counsel, not something to invent here. Treat every notice string in these repos as a placeholder.

The metrology framing stays — one definition, many independent realizations, kept honest by comparison — because it is the right intellectual model and it earns its place on the argument. It is a borrowed concept, never a claim of institutional backing.

## Repository layout (target; create as needed)

Three repositories under the `markleft-lang` GitHub org — never under a personal account. Full rationale, deferred repos and their triggers, the governance decision, and naming rules: `notes/repo-layout.md`.

**Launch Claude Code from inside a repo, not from the org folder.** The org folder is a plain directory holding three repos, so a `CLAUDE.md` there could never be committed or cloned. `tests` and `.github` carry their own thin scoped `CLAUDE.md`; this file is the canonical one and they point back to it.

One repo per layer of the normative hierarchy — definition, comparison, realizations. The namesake repo **is** the standard, which is the strongest available statement that the document is normative and not the code.

- **`markleft-lang/markleft`** — the language itself: definition, grammar, normative prose, deltas appendix. The directory containing this file; all paths in this document are relative to it. CC0-intent.

- **`markleft-lang/tests`** — conformance suite + prose corpus, the key comparison. Separate so third-party implementations can vendor it without cloning a parser and a website. CC0-intent in its entirety, no MIT code inside.

- **`markleft-lang/markleft-rs`** — the Rust realization: reference parser, canonical formatter, migrator, playground. MIT. **Does not exist yet**; create it at Phase 2 when the parser starts.

- **`markleft-lang/.github`** — org profile README (first thing a visitor sees; the elevator pitch is a question) and shared issue/PR templates. Must carry this exact name — GitHub reads the org profile from nowhere else.

```
markleft/         # this repo — the standard itself, at its root
  README.md       # what this is; states the normative boundary
  definition.md      # normative — scaffold so far, no normative text yet
  grammar/        # normative — not created yet (Phase 1)
  deltas.md       # normative — doubles as the migration guide; scaffold so far
  notes/          # non-normative memos (design challenges, layout decisions)
  CLAUDE.md       # scaffolding — not part of the released standard
  .claude/        # scaffolding — decision record, prior-art survey, conventions

tests/            # the key comparison
  examples/       # spec.txt-style paired examples
  corpus/         # prose documents authored for this project — never scraped

markleft-rs/      # Phase 2 — the Rust realization, MIT
  bindings/       # wrappers over this realization (not independent implementations)
```

**There is no `spec/` directory, deliberately.** The repository *is* the specification, so the specification sits at its root: cloning `markleft` gets you the standard, and no URL contains `markleft/spec/`. Nesting it would reintroduce the monorepo framing — the standard as one component among several — which is exactly what the layout decision removed.

The licensing boundary that a `spec/` directory would have drawn is drawn by exclusion instead, which costs nothing: **everything here is the standard except `CLAUDE.md` and `.claude/`**, which are working scaffolding, neither normative nor part of what gets released. `README.md` states that so it can be settled without reading any file. A licensing line does not need a directory to hold it.

**`markleft-<lang>` is reserved for independent implementations** — `markleft-py`, `markleft-cpp`, and so on, each a genuine realization written against the spec. A *binding* is not a realization: a thin wrapper over the Rust core via WASM or FFI belongs in `markleft-rs/bindings/`, because running the conformance suite against it compares one realization with itself, and a key comparison between a thing and itself measures nothing.

**`tests/corpus/` is original material only.** Nothing scraped, imported, quoted, or adapted — no collected READMEs, no third-party samples. Every file is contributor-authored, which keeps one licence across the whole suite. It exists to attack invariant 1 (`.claude/decisions.md` §3: prose-safety is "testable via a prose corpus in CI"), so write documents that try to break prose-safety, not ones that flatter it.

**Current state:** all three repos exist locally with commits and GitHub remotes under the `markleft-lang` org — `markleft`, `tests`, and `.github`; `markleft-rs` is not among them and is not due until Phase 2. This repo holds `README.md`, `definition.md`, and `deltas.md` — the latter two are **scaffolds**, carrying section headings and scope notes with no normative text yet — plus four memos under `notes/` and the `.claude/` scaffolding. `tests` has its `examples/` and `corpus/` directories awaiting content. This is expected at Phase 0: the definition is prose, and no code ships before Phase 1. There is consequently no build, lint, or test command to document; add a Commands section here the moment `markleft-rs` lands, and make sure it covers running a *single* conformance example — that is the unit of debugging.

## Current phase: Phase 0 — Definition & Corpus

Immediate work, in order:

1. **Draft `definition.md`** — the file exists as a scaffold: eleven section headings, a scope note for each, and a pointer to its source material, with no normative text yet. The work is writing the prose into it, in public-standard voice, because a standard has to read like one: the five invariants; the binding decisions each with rationale and rejected alternatives; Name section (four readings + copyleft lineage + the availability accounting); metrology framing (the conformance suite is a key comparison for parsers; the canonical formatter is a reference realization); licensing and copyright posture, including the explicit non-endorsement statement; governance sketch — which says the project is maintained by its authors and governed by no institution.

2. **Seed `tests/corpus/`** — original prose written to attack invariant 1: bare dollars and money amounts, snake_case identifiers, `5 * 3 = 15`, shell snippets, file globs, and lines that accidentally open with `#` or `-`. Contributor-authored only; see the corpus rule above.

3. **Prototype the block parser sketch** — enough of the prefix-decidable block algorithm to run decisions 4, 5, 6 (headings, code blocks, lists) against the corpus; they carry the highest muscle-memory risk.

Where Phase 0 sits in the ladder (full detail: `.claude/decisions.md` §9): Phase 1 grammar & spec-as-tests (PEG inlines + small-step block algorithm) → Phase 2 reference parser & specified JSON AST with source positions → Phase 3 killer tools (validator with real diagnostics, canonical formatter, migrator) → Phase 4 ecosystem (cheat sheet, playground, conformance badge, a system-prompt-sized language description for LLM emitters). Don't build ahead of the phase: the definition fixes the language before any parser encodes it.

## Self-hosting goal

Every document this project ships should be written in Markleft. This is the compiler sense of *self-hosting* — a compiler written in the language it compiles — applied to documents, and it is a stronger claim than dogfooding because it has a finish line. It cannot be claimed before a parser exists, so it is staged:

1. **Now (Phase 0) — Markleft-valid content, `.md` extension.** Every `.md` file in every repo is written to be valid Markleft that renders identically under CommonMark. This is the dogfood rule with a name and a target.

2. **Phase 3 — enforced in CI.** The validator checks every `.md` in every repo: it parses as Markleft, and its CommonMark rendering matches byte for byte. Failures are fixed, or exempted with a stated reason. This is where the claim stops being aspirational and becomes testable.

3. **Phase 4 — `.markleft` sources where they earn it.** Documents using Markleft-only features (math, decorators) cannot be valid CommonMark. Those become `.markleft`, with the canonical formatter generating `.md` companions marked do-not-edit; the site renders `.markleft` natively via WASM.

**GitHub renders `.md` and will not render `.markleft`.** That constraint does not go away, and pretending otherwise would cost us the platform that decided Markdown's adoption in the first place. So `.md` files stay. The only real question is whether they are authored or generated — and stage 3 is worth doing only for documents that genuinely need syntax CommonMark lacks. Generated files are a maintenance tax; do not pay it for documents that could simply have stayed valid in both.

**The measurement worth keeping.** The fraction of our own documents that *require* `.markleft` — that cannot be valid CommonMark — is a direct reading of how far Markleft diverges from Markdown in practice. If nearly everything can stay `.md`, decision 12 is doing its job. It is cheap to compute once the validator exists, and it recovers part of what was lost when the collected corpus was dropped: our own repos are corpus item #1, and unlike scraped material we can publish every byte of them.

## Markdown and Git conventions

Both are portable modules under `.claude/`, imported here so they stay in context. They contain no project-specific content and can be copied verbatim into any repository.

@.claude/markdown-conventions.md

@.claude/git-conventions.md

Two project-specific riders sit on top of them:

- **Dogfooding is separate from source formatting.** The Markdown conventions govern how the *source* is formatted and apply to every `.md` file here, this one included. Dogfooding governs which Markdown *constructs* are permitted and applies to spec documents — see the dogfood bullet below.

- **Cross-repository pairing.** A spec change and its conformance examples land as paired pull requests across `markleft` and `tests`, per `notes/repo-layout.md`. That is a workflow rule of this project, not a general git convention, which is why it is not in the portable module.

## Working conventions for Claude Code sessions

- Consult `.claude/decisions.md` before revisiting anything settled there; if new evidence (corpus failures, migrator change reports, implementer bug reports) challenges a decision, don't silently change it — write the challenge into a `notes/` memo and flag it.

- **The document is normative; the suite is the key comparison.** The spec — grammar included — defines the language; the conformance suite checks it, and is mandatory but subordinate. When they disagree the document governs and the test is presumed to be the bug — *unless* the test is evidence the spec itself needs revising, which is a spec change through the decision record, never a quiet edit. No behavior may exist only in the suite. Full rule and the three-way triage: `notes/normative-hierarchy.md`.

- A spec change and its examples land as **paired pull requests** across repos — opened together, merged together, neither alone. `tests` records the spec revision it targets; the spec is never tagged ahead of the suite that exercises it. See `notes/repo-layout.md`.

- Dogfood: write all project prose so it's valid under our own rules (no underscore emphasis, fenced code only, no raw HTML) — our own repo is the first thing that has to survive them. Note the one asymmetry: **escape dollars in `.md` files even though Markleft never needs it** — bare `$` is always literal here, but GitHub's math pass will pair them on render, which is the founding exhibit happening to us. (This file uses standard Markdown for Claude Code's benefit; spec documents should dogfood.)

- Prefer boring, explicit rules over clever heuristics — cleverness is how Markdown got here.

- Humor budget: the "What's left?" pun and the extension graveyard are canon; everything else in normative text stays sober.

- Attribute inheritances explicitly in spec prose — CommonMark (spec-as-tests methodology, byte-compatibility baseline), djot (linear-time architecture, uniform escaping, attributes — reduced here to decorators, bracketed emphasis; **not** its raw-content design, which decision 7 removes), MyST/rST (the directive concept disciplined into a closed extension point, never the content-generating directive), Markdoc (schema validation), Typst (error-message bar), AsciiDoc/Eclipse (TCK concept). Full list: `.claude/landscape.md`. Most of our syntax is djot-vetted; saying so is both accurate and disarming.

## Open checklist (verify/act early in session when relevant)

Master copy is `.claude/decisions.md` §10, mirrored in `.claude/plan.md` — update all three when an item closes, or they drift. Items below are the ones with an external dependency (registry, registrar, counsel) that a session may be able to advance opportunistically.

- [ ] Claim npm scope `@markleft`

- [ ] Claim crates.io `markleft` (or `markleft-core`)

- [ ] Acquire domain (markleft.org / markleft.ca)

- [ ] Legal: trademark search (CIPO/USPTO/EUIPO). **Licence settled on CC BY 4.0**; what remains for counsel is (a) the prior question of whether this is Crown copyright at all — s.12 and s.13(3) turn on direction/control and course of employment, not employment status — and (b) exact notice wording. Does not imply institutional oversight of the project itself.
