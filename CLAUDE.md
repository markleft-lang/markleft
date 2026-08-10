# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

**Markleft** is a formally specified, ambiguity-free successor to Markdown: Markdown's size and prose-like source, with a formal grammar, an executable conformance suite, and exactly one parse for every input. **Independent project, maintained by its authors — no institutional steward, sponsor, or endorser.** Crown copyright attaches by operation of law, which is not sponsorship; see Licensing below, and never imply otherwise anywhere. GitHub org: `markleft-lang`. See `.claude/decisions.md` for the full decision record and rationale — read it before proposing design changes. See `.claude/plan.md` for the running plan — current work, parked ideas, errands, and open questions in plain language; add to it as things come up. See `.claude/grammar.md` for the running rule list — every construct that has operational meaning, serially numbered, each with the plain-text writing it collides with and a severity; **check a proposed decision against it before recording the decision, and add the new rule in the same session.** Its reserved-position table is first in the file deliberately: it is the at-a-glance answer to "what is safe to type", so read it before the rules and update it in the same edit as any new rule. It is the working page behind invariant 1's "the set of structurally meaningful positions fits on one page", and it seeds the Phase 1 BNF. See `.claude/landscape.md` for the prior-art survey (djot, CommonMark, MyST, Quarto, Markdoc, Typst, AsciiDoc, etc.) — consult it when writing the definition's prior-art section or evaluating features other languages have.

The name: markup went up, markdown went down, Markleft moved sideways — and the language is *what's left* after fifteen years of curation. When someone asks "what's left?", that question is the elevator pitch. A fourth reading arrived with decision 20 and is now load-bearing: **every construct's mark sits to the left of its content**, blocks marking the left of a line and inlines the left of a span. A retronym — the name came first — and it produced the tagline:

> **Everything that is not prose leaves a mark.**

Read it backwards for the guarantee: **no mark, therefore prose.** That is invariant 1 from the reader's side and the cardinal rule at once. It was not true of the language before decision 20, because flanking rules decided whether `*` was a mark by looking at what surrounded it. Note the pun is structural: *leaves a mark* and *Markleft* are the same two words in different tenses.

## Non-negotiable invariants

Every contribution holds all five. A proposal that breaks one cannot land as it stands — changing that means amending the invariant itself, in the open, with the argument written down.

1. **Prose-safety:** natural-language prose renders verbatim; money, snake_case, `5 * 3 = 15`, shell snippets are safe *by construction*, never by heuristic. **Since decision 20 this holds with no residue** — flanking rules were the last heuristic in the language, and they are gone. There is no rule anywhere that decides a character's meaning by looking at what surrounds it.

2. **Five-minute property:** the complete core fits on one reference card; no rule has an "unless" clause.

3. **One-meaning property:** every input has exactly one parse; the spec is executable (grammar + tests), not interpretable prose.

4. **Linear-time property:** single pass, prefix-decidable blocks, no backtracking, no pathological inputs.

5. **Cardinal rule:** you can always read the entire document in plain text, with no compiler, viewer, or extension; nothing renders that the source does not already say. **Enabling, not restrictive** — the minimal set that lets capability live in tooling and be built cleanly. Detail: decision 15.

**Non-goals (definition-level, from `.claude/landscape.md`):** Markleft is a language, not a toolchain and not a document-preparation system. No templating, no executable code cells, no scripting, no page/layout awareness, no output-format directives. Those make a different product (Quarto, Typst, Quarkdown) and every one of them front-loads complexity that the five-minute property cannot absorb. Feature requests in that direction are out of scope, not merely deferred. Decision 15 turns this from a scoping statement into a syntax rule: **nothing renders that is not in the source**, so no TOC directive, no auto-numbering, no transclusion — you can always read the whole document in plain text, with no compiler or viewer.

**Not a richer Markdown for its own sake — but not an austerity project either.** Markleft *does* add: math in the core, decorators, pipe tables, several adopted from djot. The rule is that every addition pays for itself: it removes an ambiguity or fills a genuine gap, **and costs nothing in plain-text clarity** — a construct that makes ordinary unmarked prose read worse does not clear the bar, however much it offers elsewhere, because prose-safety is invariant 1 and richness is not an invariant at all.

**The five-minute property is the budget.** Most of the binding decisions remove something; a few add; the total still fits on one reference card. So "Markleft could also do X" is never an argument by itself — the argument must be that X is worth what it costs every reader who never uses it. When weighing a djot feature we have not adopted (smart punctuation, definition lists, generic containers), that is the question to ask, and `.claude/landscape.md` records why djot is the right place to shop.

## Core language decisions

Summary only — rationale and the alternatives set aside live in `.claude/decisions.md` §4. Numbering is normative and matches that section: cite decisions by number ("challenge to decision 6"), here, in commits, and in `notes/` memos.

1. Bare `$` is ALWAYS literal text. Math is core: `` `x=y`{math} `` inline, ```` ```math ```` block. Math content is verbatim — TeX backslashes survive. "Core" = three guarantees: `$` is never syntax, the construct exists at both sizes with verbatim content, the label reaches the AST intact. `math` is a conventional label, not a reserved word (decision 9).

2. Emphasis is braced: `*{em}`, `**{strong}`, `***{both}`. **No paired form and no flanking rule anywhere in the language** — flanking was prose-safety by heuristic where invariant 1 promises by construction, and it was the last such place. Runs of 1–3 are defined; 4+ is text. Underscore is not emphasis; its one use is decision 18's subscript. Typing cost: `+1` char for emphasis, free for strong, `−1` for both.

3. **Escaping is a range: `\{…}`.** A backslash followed by anything but `{` is an ordinary backslash. The old "escape ANY character, no exception list" rule read perfectly and was the only rule that made writing *worse* than CommonMark — it rendered `C:\Users\frederic` as `C:Usersfrederic`. Content inside a range balances, so `\{a {b} c}` needs no lengthening. The one non-brace operand is a line ending (decision 13).

4. ATX headings only, one closed form: one to six `#` in the first column, a space, then non-empty text; any line that doesn't match is text. No setext, no closing `#` run, no indentation, no empty heading.

5. Fenced code only. No indented code blocks. **Backtick runs are maximal, and a fence closes on a run of exactly the opening length** — the same counting rule as a verbatim span, where CommonMark uses two. **The run length is the only escape inside verbatim**, decision 3's backslash being switched off there, which is what "one or more" and "three or more" are for; the minimum of three is a separate rule answering prose-safety. Cost: an over-long closing run does not close, and the validator says so.

6. Strict list rules: `-` is the only bullet marker and `1.` the only ordered marker; content-column alignment; no lazy continuation. Adjacent items are one list, and a list ends only at a non-list block — blank lines never split one, they only make it loose. **"One marker per list" is deleted rather than reworded**: `*`, `+`, and `1)` existed only so a marker switch could separate two touching lists, so the rule left with them. `+` is now free everywhere. Ordered lists renumber from the first marker's value. **Alphabetic and Roman markers stay out** — `A. Smith`, `J. R. R. Tolkien` put a severity-3 collision at line start, and a numbering style written into the source is what decision 9 removed classes for. *Highest muscle-memory risk, with 4 and 5.*

7. No raw passthrough at all — no raw block, no raw span, no `=format` decorator. A Markleft document is inert by construction. Raw HTML would be a decision 15 hole (`<iframe>` is transclusion).

8. Pipe tables, strictly grammarized. **Any line beginning with `|` opens a table** — which closes the block layer's last lookahead and makes the leading pipe mandatory by construction. Cells are separated by `|`; **a `|` alone on a line opens a block cell** holding paragraphs, lists, or fences. Column count comes from the first row; a rule row fixes alignment and its position says whether there is a header — first line means none. **A table ends at a blank line after a line beginning with `|`**; the closing mark, a lone `|` before a blank line, opens no cell. GFM tables all still work. **No width hints** — `notes/table-width-hints.md`.

9. Decorators — one vocabulary, two positions: ```` ```math ````, ```` ```math #eq1 ````, ```` ```#snippet-3 ```` on fences; `` `x`{math #eq1} `` inline. Two token shapes, **each at most one, no multiplicity exception anywhere**: bare word (format label, kebab-case ok) and `#id` (an anchor — navigation, not style; duplicate ids across a document are a lint check, not a parse error). **There is no `.class`** — it was the only construct whose purpose was to let rendering vary, it strips to nothing on platforms that sanitize CSS, it balloons the source, and it invites mixing styling with content. No `key=val`, no `=format`. Token character set: Unicode restricted by exclusion only — whitespace, control characters, `{` `}`, and the backtick; parentheses left the list 2026-08-09. **No word is ever reserved:** labels are opaque, carried into the AST, and no decorator word can affect the parse. Meanings (`math`, `rust`) are convention, held in a non-normative appendix and the validator's list; unknown words parse as plain verbatim and warn on lint. **Bare ```` ``` ```` and `` `…` `` mean verbatim plain text, period** — that is a definition, not a default. No `text` label: nothing rules it out (nothing is reserved), and nothing blesses it — the validator calls it redundant and the formatter strips it.

10. Tabs are not structural (validator warning + auto-fix).

11. **Markdown's block layer is kept; its inline layer is replaced.** No smart punctuation. The muscle-memory clause was always conditional on Markdown not being broken, and the line falls on the block/inline boundary: a Markdown writer's fingers are right about all six block constructs and wrong about every inline one. Backtick verbatim is the only inline construct that survives — challenged on keyboard-accessibility grounds and kept, `notes/backtick-verbatim-challenge.md`.

12. **Markleft is NOT Markdown-compatible, by construction and on purpose.** The old rule said match CommonMark *wherever unambiguous and harmless*, and the replaced inline constructs are neither — so the clean break is what that rule licensed, not a reversal of it. Near-compatibility is what produced flavour drift; a clean break makes "does my document survive this platform" answerable. The two extensions (§6) are the instrument. `deltas.md` is a **translation guide**, not a migration guide.

13. Hard breaks are explicit: `\` before a line ending is the only one. Trailing-space breaks are removed, and trailing whitespace is never significant.

14. Unicode is the character set, UTF-8 the encoding; every code point is text and content is never normalized. *The four riders — invalid UTF-8, leading BOM, anchor/label matching, what "column" means outside ASCII — are answered in `definition.md` §4.1 and held as Appendix D items 1–4 pending confirmation. The decorator character-set rider closed 2026-08-07 (decision 9).*

15. No generated content — the source is the whole document. No TOC directive, auto-numbering, index, transclusion, or variable substitution. **Cardinal rule: you can always read the entire document in plain text, with no compiler, viewer, or extension.** The source need not *look* like the rendered form (`**bold**` is not bold); it must be *semantically equivalent* to it, up to format. Test for any construct: is it changing the presentation of written content, or manufacturing content? Presentation instructions stay open; manufacturing is closed. **Back half:** rendering is lossy on marks and lossless on content — a decorator disappears into the typography that re-expresses it, so every format word must name something a renderer visibly distinguishes.

16. Images are core: `![alt]{src}`, or `!{src}` with no alt. Legal because **an image adds no text**: decision 15 governs the document's text, so "no transclusion" there means no *textual* transclusion. Alt is optional; empty alt is a lint warning, never an error. **The memorable rule: `@[…]` links to the target, `![…]` shows it instead — the `!` is a presentation switch, not a different construct**, and under decision 20 the two forms differ by exactly one character.

17. Anchors are positional: `#{id}` valid **anywhere**, marks a point, not an attachment — so no paragraph exclusion and no adjacency rule. **An anchor marks where the reader arrives** (lead for anything longer than a line, trail on a heading). Whitespace around it is optional and insignificant. **References need no new syntax:** `@[text]{#id}`, `@[text]{file.md#id}`, and `@[text]{https://…#id}` are one construct — a link to a URL fragment. Markleft's xref contribution is *removing the slug heuristic*, not adding a feature. **No reference links, no link reference definitions, no autolinks** (removed under decision 20, recorded here): the shortcut form was the only construct with action at a distance, where `[sic]` became a link because a definition existed elsewhere in the document. No bare sigil: it would have to generate its own text — decision 20 answered the prose-safety half of that objection and left the generated-text half standing. Validator stays single-document; a target with a path or host is a URL and is not validated.

18. Superscript `^{…}` and subscript `_{…}` are core. **A sigil means nothing unless the very next character is `{`** — so `2^32`, `[^0-9]`, `ISO_8601`, `file_2.txt`, and `snake_case` are all still text with no escape, and there is no braceless form (a one-digit cap would render `2^32` as 2³2 — wrong, not merely limited). **Baseline offset and size only: no italics, no math spacing, no interpretation.** That is the definition, and it is why this is not a second way to write math — `E=mc^{2}` and `` `E=mc^2`{math} `` render differently. Boundary: *these position content and interpret nothing; anything needing interpretation is `math`.* Content is inline, so `x^{*{n}}` emphasizes because the author asked. **This decision's sentence turned out to be the whole language's organizing principle** — see decision 20; it was recorded here as "consistency lost to legibility, deliberately and once", and it was neither once nor a loss.

19. **No thematic break.** `---`, `***`, `___` reserve nothing — paragraph text, or a bullet item where it takes that form (`- - -`). Removed because it has **no plain-text meaning**: a hyphen run says something changes and never says what, where a heading says which. Its rendering is a style choice besides, colliding with the thin rule most house styles already draw under an h2. **Where a break was wanted, write the heading it stood in for, or a second document.** Returns `*` and `_` to the free line-start column, and deletes R8's only precedence exception. Cost: a real delta, loud not silent, and the one delta the migrator reports rather than converts.

20. **Every inline construct opens with a sigil and a brace.** Ten sigil runs, one shape: `*{}`, `**{}`, `***{}`, `^{}`, `_{}`, `#{}`, `@{}`, `!{}`, `\{}`, `` `{} `` — extended by `[` for the link and image forms that carry their own text (`@[…]{…}`, `![…]{…}`). **Three findings carried it, not consistency:** decision 3 was less prose-safe than CommonMark (`C:\Users` mangled); decision 11 never protected emphasis (the most broken construct CommonMark has); and braced emphasis removes invariant 1's last heuristic. **The reference card is three sentences** — blocks decided by position in the line, inlines by a sigil then `{` (or `[` when carrying text), verbatim by matching backtick runs (the one exception, and it explains itself: uninterpreted content cannot be balance-counted). **One escape mechanism everywhere: when a delimiter appears in your content, lengthen the delimiter.** Single braces work whenever content balances. **Ledger:** `@` spent at severity 1; `\` 3→1, `*` 2→1, `[`/`]` and `{`/`}` 2→1, `(`/`)` and `<` to 0. Seventeen of thirty-two ASCII punctuation characters free; `-` the only severity-3; line-start column holds exactly the six block markers. Costs: the clean break (decision 12), `#{` collides with Ruby interpolation, `*{` with shell brace expansion, and braces are AltGr on several layouts.

21. **Task lists are core, in Markdown's form, strictly:** `- [ ]` and `- [x]` — a bullet item whose content opens with the three-character state box, then a space. Lowercase `x` only, bullet items only, exact spacing; anything else is item text (`[X]` is a validator warning with a formatter fix). **The first capability that fails the routing test**, so the grammar carries it: Unicode ☐/☑ fails on typability, and a renderer special-casing literal `[x]` would be bolted-on post-processing — the founding exhibit's architecture. `-{x}` and `-[x]` not adopted: sigil-then-brace is the inline layer's rule and the block layer holds no brace; the no-space form would turn every GFM-trained hand's `- [x]` into literal text. `[` spends one narrow position at severity 1. A clickable checkbox is a tool writing one character into the source.

22. **Strikethrough is core: `~{…}`** — a single tilde immediately before `{`; a run of two or more tildes is text, so GFM's `~~struck~~` is literal here and the migrator rewrites it mechanically. One tilde, not two: the emphasis ladder is the only reason any sigil runs past one, and strikethrough has one meaning, so it joins the run-1 family (`^`, `_`, `#`, `@`, `!`, `\`) — a lone `~~{…}` would be the language's only inert minimal run, an "unless" clause with no ladder behind it. Typographic strike only, decision 18's boundary. Ledger: `~` spent at severity 1 (shell's `~{a,b}` expansion, answered by a verbatim span); sixteen of thirty-two ASCII punctuation characters now free; line-start column untouched. Closes the strikethrough half of Finding 5 / Appendix D.14, leaving entity references; D.5's tilde-fence answer now rests on the one-way-to-do-it argument alone.

23. **Asides are core: `<` is the converse of `>`** — a line beginning `<` then a required space is a line of an aside, `<` alone its blank line; a container nesting and holding any blocks, the block quote mirrored. Plain-text semantics: `>` presses harder than prose (quoted matter, standing out), `<` presses less (liftable with little impact) — meaning stated in the source, the test decision 19's thematic break failed. **Rendering open by design: sidebar, margin note, smaller type, or a disclosure fold — folding is one rendering, not the construct, so the mark means something in print**; retires decision 7's `<details>` casualty natively while the raw-HTML hole stays closed. The one asymmetry with `>` is the required space, which keeps pasted `<div>`/`<!DOCTYPE` lines text; no title line, no type vocabulary (`< note` would be a class — decision 9; "Note:" is text). Ledger: `<` respent at severity 1 a day after decision 20 freed it; fifteen of thirty-two free; **the line-start column grows to seven** — the six-marker sentence in decisions 20–22 is superseded knowingly.

## File extensions

Both forms are DECIDED (`.claude/decisions.md` §6). Canonical pair, exactly two: `.markleft` (long form — documentation-canonical, the collision-proof long-term anchor) and `.lf` (short form — the `.md` two-letter echo and the line-feed wink). Never use: `.lft` (runner-up, reserve fallback only — not a live option), `.left`, `.mklf` (retired), `.ml`, `.mll` (OCaml/ocamllex), `.mlf`, `.mlt`, `.mlk`, `.mkt`, `.mf`.

## Licensing & copyright (apply to every new file)

**Crown copyright applies by operation of law, not by choice and not as sponsorship.** The author is a Canadian federal public servant, so Copyright Act s.12 vests copyright in the Crown whatever anyone would prefer. It carries no endorsement, no review authority, and no governance role over this project. Never phrase a notice so that a reader could infer a department backs, owns, supervises, or vouches for the language.

- **Spec & conformance suite: CC BY 4.0** *(settled 2026-08-07)* — the most open CC licence short of CC0: attribution only, no share-alike, no non-commercial, no no-derivatives. Chosen because OGL-Canada 2.0, the federal open-licensing instrument, grants substantively the same rights. Mark files "Crown copyright, Canada — 2026. Licensed CC BY 4.0." **Never write the monarch's style into a notice**, and never write "Government of Canada" either: the first is gratuitous, the second reads as a government publication and reimports the sponsorship inference. "Crown copyright" is the term of art, names no person and no institution, and is what a reuser searches for. **Public files state the position; they never show the reasoning or hedge it** — the wording is still provisional pending counsel, but that fact lives here and in the decision record, not in a notice. CC0 remains the stated *intent*, blocked on authority rather than law — a public servant cannot dedicate Crown material on the Crown's behalf. **Do not claim OGL-Canada declares CC BY compatibility; it does not** — that clause is the UK OGL's.

- All code: MIT.

- Exact notice wording is a question for counsel, not something to invent here. Treat every notice string in these repos as a placeholder.

The metrology framing stays — one definition, many independent realizations, kept honest by comparison — because it is the right intellectual model and it earns its place on the argument. It is a borrowed concept, never a claim of institutional backing.

## Repository layout (target; create as needed)

Three repositories under the `markleft-lang` GitHub org — never under a personal account. Full rationale, deferred repos and their triggers, the governance decision, and naming rules: `notes/repo-layout.md`.

**Launch Claude Code from inside a repo, not from the org folder.** The org folder is a plain directory holding three repos, so a `CLAUDE.md` there could never be committed or cloned. `tests` and `.github` carry their own thin scoped `CLAUDE.md`; this file is the canonical one and they point back to it.

One repo per layer of the normative hierarchy — definition, comparison, realizations. The namesake repo **is** the standard, which is the strongest available statement that the document is normative and not the code.

- **`markleft-lang/markleft`** — the language itself: definition, grammar, normative prose, deltas appendix. The directory containing this file; all paths in this document are relative to it. CC BY 4.0, with CC0 the stated intent (blocked on authority, not law).

- **`markleft-lang/tests`** — conformance suite + prose corpus, the key comparison. Separate so third-party implementations can vendor it without cloning a parser and a website. CC BY 4.0 in its entirety, no MIT code inside; CC0 the stated intent.

- **`markleft-lang/markleft-rs`** — the Rust realization: reference parser, canonical formatter, migrator, playground. MIT. **Does not exist yet**; create it at Phase 2 when the parser starts.

- **`markleft-lang/.github`** — org profile README (first thing a visitor sees; the elevator pitch is a question) and shared issue/PR templates. Must carry this exact name — GitHub reads the org profile from nowhere else.

```
markleft/         # this repo — the standard itself, at its root
  README.md       # what this is; states the normative boundary
  LICENSE         # CC BY 4.0 legal code, verbatim — GitHub reads this
  definition.md      # normative — scaffold so far, no normative text yet
  grammar/        # normative — not created yet (Phase 1)
  deltas.md       # normative — the translation guide; scaffold so far
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

The licensing boundary that a `spec/` directory would have drawn is drawn by exclusion instead, which costs nothing: **everything here is the standard except `CLAUDE.md` and `.claude/`**, which are working scaffolding, neither normative nor part of what gets released; `LICENSE` is Creative Commons' own text, reproduced verbatim. `README.md` states that so it can be settled without reading any file. A licensing line does not need a directory to hold it.

**`markleft-<lang>` is reserved for independent implementations** — `markleft-py`, `markleft-cpp`, and so on, each a genuine realization written against the spec. A *binding* is not a realization: a thin wrapper over the Rust core via WASM or FFI belongs in `markleft-rs/bindings/`, because running the conformance suite against it compares one realization with itself, and a key comparison between a thing and itself measures nothing.

**`tests/corpus/` is original material only.** Nothing scraped, imported, quoted, or adapted — no collected READMEs, no third-party samples. Every file is contributor-authored, which keeps one licence across the whole suite. It exists to attack invariant 1 (`.claude/decisions.md` §3: prose-safety is "testable via a prose corpus in CI"), so write documents that try to break prose-safety, not ones that flatter it.

**Current state — the language is at 0.1.0; this repo is tagged through v0.1.1.** All three repos exist locally with commits and GitHub remotes under the `markleft-lang` org — `markleft`, `tests`, and `.github`; `markleft-rs` is not among them and is not due until Phase 2. This repo holds `README.md`, `definition.md`, and `deltas.md`, plus six memos under `notes/` and the `.claude/` scaffolding. `definition.md` is a **full draft** rewritten for decision 20 — twelve sections and four appendices, the language in prose in §4 and the decisions with rationale in §5; Appendix D lists fourteen open points, five answered for the first time and needing confirmation, two raised by decision 20. `deltas.md` is still a **scaffold**: the two delta tables are complete as to which decisions diverge, the per-delta detail is not written. `tests` has its `examples/` and `corpus/` directories awaiting content. No build, lint, or test command exists; add a Commands section here the moment `markleft-rs` lands, and make sure it covers running a *single* conformance example — that is the unit of debugging.

**Versioning.** Tags are waypoints, not per-commit. `v0.0.1`–`v0.0.8` cover the scaffolding pass retroactively; `v0.1.0` is decision 20. The bands: **`0.x.0`** when the language changes, **`0.x.y`** for everything else, **`1.0.0`** when the definition is normative and a realization passes the suite. For a specification the public API is *the language* — what parses, and to what — which is what makes the bands decidable. Merge exploratory work with `--no-ff` so the bubble stays visible and droppable.

## Current phase: Phase 0 — Definition & Corpus

Immediate work, in order:

1. **Review and settle `definition.md`** — the full draft is written and rewritten for decision 20. What remains is the author's review, and closing Appendix D: items 1–4 are answered for the first time and need confirming (invalid UTF-8, leading BOM, code-point identity for matching, code-point count for "column"), and D.5 now states its as-drafted answer (tilde fences not inherited) pending the same confirmation; items 6–14 are unanswered except D.6, closed by decision 19. The newest are D.12 (may a brace group span a line ending) and D.13 (is `**{…}` worth having), both raised by decision 20, and D.14 (found absent during its review; its strikethrough half is closed by decision 22, leaving HTML entity references). Anything confirmed goes into `.claude/decisions.md` as an amendment, not silently into the prose.

2. **Seed `tests/corpus/`** — original prose written to attack invariant 1. Decision 20 changed what the corpus should attack: the old targets (bare dollars, `snake_case`, `5 * 3 = 15`, file globs) are now safe by construction and worth one document each as regression evidence. The live targets are the **doublet openers** — `#{` in Ruby and Elixir interpolation, `*{` in shell brace expansion, `\{` in TeX — plus the seven block markers at line start, which are where the remaining severity 2 and 3 collisions all live. Contributor-authored only.

3. **Prototype the block parser sketch** — enough of the prefix-decidable block algorithm to run decisions 4, 5, 6 against the corpus. Add an inline sketch alongside it: R30's brace group is the whole of Layer 2, and the corner worth testing first is an inline-content group containing a literal-content group whose content holds an unbalanced brace.

Where Phase 0 sits in the ladder (full detail: `.claude/decisions.md` §9): Phase 1 grammar & spec-as-tests (PEG inlines + small-step block algorithm) → Phase 2 reference parser & specified JSON AST with source positions → Phase 3 tooling (validator with real diagnostics, canonical formatter, migrator) → Phase 4 ecosystem (cheat sheet, playground, editor highlighting, conformance badge, a system-prompt-sized language description for LLM emitters). Don't build ahead of the phase: the definition fixes the language before any parser encodes it.

## Self-hosting goal

Every document this project ships should be written in Markleft — the compiler sense of *self-hosting*, applied to documents. It cannot be claimed before a parser exists.

**Decision 20 collapsed the ladder from three stages to one, and this section is rewritten rather than annotated because the old stages are not merely delayed, they are impossible.** The former stage 1 was "every `.md` file is valid Markleft that renders identically under CommonMark", and stage 2 made that a CI check. Under a clean break, a document containing a link is one language or the other. Every document here contains a link.

**What replaces it.** Project `.md` files are **Markdown, and not written in Markleft**. That is stated plainly in `README.md` rather than left to be discovered, and it is honest rather than embarrassing: nobody can dogfood a language that has no parser. When the reference parser and canonical formatter land (Phases 2 and 3), documents move to `.markleft` sources with generated `.md` companions marked do-not-edit, and CI checks that the pair is in sync. That is one stage, it arrives at Phase 3, and it is the only one.

**GitHub renders `.md` and will not render `.markleft`.** That constraint does not go away, and pretending otherwise would cost us the platform that decided Markdown's adoption in the first place. So `.md` files stay; the question is only whether they are authored or generated, and until Phase 3 they are authored.

**The measurement that went with the old ladder is gone, and the loss should be recorded rather than mourned.** The fraction of our own documents requiring `.markleft` was to be a direct reading of how far Markleft diverges from Markdown in practice. That answer is now known in advance — all of them — so the number carries no information. The divergence it was built to detect was decided rather than discovered, which is the honest description of what decision 20 did.

## Markdown and Git conventions

Both are portable modules under `.claude/`, imported here so they stay in context. They contain no project-specific content and can be copied verbatim into any repository.

@.claude/markdown-conventions.md

@.claude/git-conventions.md

Two project-specific riders sit on top of them:

- **Dogfooding is separate from source formatting.** The Markdown conventions govern how the *source* is formatted and apply to every `.md` file here, this one included. Dogfooding governs which Markdown *constructs* are permitted and applies to spec documents — see the dogfood bullet below.

- **Cross-repository pairing.** A spec change and its conformance examples land as paired pull requests across `markleft` and `tests`, per `notes/repo-layout.md`. That is a workflow rule of this project, not a general git convention, which is why it is not in the portable module.

## Working conventions for Claude Code sessions

- Consult `.claude/decisions.md` before revisiting anything settled there; if new evidence (corpus failures, migrator change reports, implementer bug reports) challenges a decision, don't silently change it — write the challenge into a `notes/` memo and flag it.

- **Watch the voice: authoritative language earns its place when an objective ruling is doing the work — a grammar rule, a program, a definition — and not when a person is.** The test and the substitution corpus are in `notes/voice-and-authority.md`; reach for it whenever a sentence wants to say *forbids*, *rejected*, *allowed*, or *required*. This project is built to be community-owned, and the prose says so or it doesn't.

- **The document is normative; the suite is the key comparison.** The spec — grammar included — defines the language; the conformance suite checks it, and is mandatory but subordinate. When they disagree the document governs and the test is presumed to be the bug — *unless* the test is evidence the spec itself needs revising, which is a spec change through the decision record, never a quiet edit. No behavior exists only in the suite. Full rule and the three-way triage: `notes/normative-hierarchy.md`.

- A spec change and its examples land as **paired pull requests** across repos — opened together, merged together, neither alone. `tests` records the spec revision it targets; the spec is never tagged ahead of the suite that exercises it — a rule binding from the suite's first release onward, since before content exists there is nothing that can lag. See `notes/repo-layout.md`.

- **Do not dogfood. Project `.md` files are Markdown.** Decision 20 ended the dogfood rule: a document containing a link is Markleft or Markdown and cannot be both, and GitHub renders only the second. Write ordinary CommonMark — `[text](url)`, `*em*`, `![alt](src)` — so these files render where they are read. Writing Markleft syntax into a `.md` file produces a document that renders as literal punctuation, which is worse than not dogfooding. Keep the one asymmetry that still bites: **escape dollars in `.md` files**, because GitHub's math pass pairs them on render, which is the founding exhibit happening to us. When a document quotes Markleft syntax, put it in a fence or a verbatim span.

- **The public status board is the phase checklist in the `.github` repo, at `profile/README.md`, and it is the single source of truth.** It restates the roadmap in `.claude/decisions.md` §9; this repo's `README.md` links to it and must never grow a second copy. The board is org-level because the phases are: Phase 1 lands as paired pull requests across two repos, and Phase 2 creates a third. Check a phase off when it completes, and add or remove phases there in the same edit that changes §9 — a stale board is worse than no board, because it is the first thing a visitor reads.

- Prefer boring, explicit rules over clever heuristics — cleverness is how Markdown got here.

- Humor budget: the "What's left?" pun and the extension graveyard are canon; everything else in normative text stays sober.

- Attribute inheritances explicitly in spec prose — CommonMark (spec-as-tests methodology, and the block layer decision 11 keeps), djot (linear-time architecture, attributes — reduced here to decorators, and the uniform-escape principle — kept as delimiter lengthening after decision 20 retired the backslash form; its bracketed emphasis was the stepping stone to the braced form; **not** its raw-content design, which decision 7 removes), MyST/rST (the directive concept disciplined into a closed extension point, never the content-generating directive), Markdoc (schema validation), Typst (error-message bar), AsciiDoc/Eclipse (TCK concept). Full list: `.claude/landscape.md`. Most of our syntax is djot-vetted; saying so is both accurate and disarming.

## Open checklist (verify/act early in session when relevant)

Master copy is `.claude/decisions.md` §10, mirrored in `.claude/plan.md` — update all three when an item closes, or they drift. Items below are the ones with an external dependency (registry, registrar, counsel) that a session may be able to advance opportunistically.

- [ ] Claim npm scope `@markleft`

- [ ] Claim crates.io `markleft` (or `markleft-core`)

- [ ] Acquire domain (markleft.org / markleft.ca)

- [ ] Legal: trademark search (CIPO/USPTO/EUIPO)

- [ ] Legal: the prior question and the notice. **Licence settled on CC BY 4.0**; what remains for counsel is (a) whether this is Crown copyright at all — s.12 and s.13(3) turn on direction/control and course of employment, not employment status — and (b) exact notice wording. Does not imply institutional oversight of the project itself

- [ ] Give `tests` and `.github` their own GitHub About boxes (needs web UI or `gh`); `markleft`'s is set — whether to adopt the exact description and six topics parked in `.claude/plan.md` is an editorial call
