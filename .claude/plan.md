# Plan

Where the project is, what comes next, and everything parked along the way. Plain language on purpose — nothing here needs a spec background to read.

**What this file is not.** It is not the decision record, which is `.claude/decisions.md` and holds *why* things were decided. It is not the rule list either, which is `.claude/grammar.md` and holds *what each construct takes away from plain text* — check a new decision against that file before recording it. It is not normative, not part of the released standard, and nothing here binds anything. When an idea here turns into a decision, it moves to the decision record and comes off this list. Keep it scruffy and current rather than tidy and stale.

## Right now — Phase 0, v0.1.0

**Decision 20 landed on 2026-08-09 and replaced the entire inline layer.** One rule now covers it: a sigil means nothing unless the very next character is `{`. Links, images, emphasis, anchors, and escapes all changed; reference links, their definitions, and autolinks are gone. The consequence a reader needs first is that **Markleft is no longer Markdown-compatible, deliberately** — see decision 12. `definition.md`, `deltas.md`, `README.md`, `CLAUDE.md`, and the grammar are all rewritten to match, and the exploration that produced it sits in the history as its own merge bubble.

Two things stand between us and a stable language. Everything else is downstream.

- **Settle the definition.** `definition.md` is a full draft — twelve sections and four appendices, the language in prose in §4 and the twenty decisions with rationale in §5. What is left is the author's read-through and Appendix D, which lists every point it leaves open. Five it answers for the first time and needs confirmed; eight remain open, two of them raised by decision 20; one is closed with its number kept. Confirming any of them is a decision, so it goes in the record before it goes in the prose.

- **Fold decisions 21 through 25 into the definition.** Task lists (`- [x]`), strikethrough (`~{…}`), asides (`< `), and comments (`// `), all adopted 2026-08-10, postdate the definition draft, so `definition.md` §4 and §5 and the reference card do not mention them yet. Add the four constructs during the settling pass — the aside and the comment amend §4.3.7's closed block-opener list, the same-day amendment requiring the quote's space amends §4.3.4, and decision 25's rider lands on decision 15's section: the document is the source minus its comments. Give the corpus a target for each new collision: a bullet item quoting a literal `[x]` box, shell tilde-and-brace expansion (`~{alice,bob}/bin`) quoted in prose, technical paragraphs opening with spaced comparisons (`< 5 ms`, `> 5 ms`), and a pasted C-family comment line (`// TODO`), the one that vanishes. Decision 24's one-line-per-block recommendation also lands in the definition's authoring guidance, non-normative, in the same pass — and the corpus should include one deliberately hard-wrapped document, since wrapped sources stay well-formed and their wrap-point collisions are exactly what the recommendation exists to avoid.

- **Write the prose corpus** (in the `tests` repo, `corpus/`). Original prose written specifically to *break* prose-safety. **Decision 20 moved the targets**: money amounts, `snake_case`, `5 * 3 = 15`, and file globs are now safe by construction and are worth one document each as regression evidence rather than as attacks. What is actually attackable is the set of doublet openers — `#{` in Ruby and Elixir interpolation, `*{` in shell brace expansion, `\{` in TeX — and the eight block markers at line start, which is where every remaining collision above severity 1 lives. Contributor-written; nothing scraped or borrowed.

The corpus matters more than it looks: it is the only thing that produces *evidence* rather than reasoning. The block decisions carry real risk of annoying people who already know Markdown — headings, code blocks, lists — and the inline layer now carries more than all of them together.

## What comes after

The ladder, in order. Nothing gets built ahead of its rung; the definition fixes the language before any parser encodes it.

1. **Grammar and tests** — the formal grammar, plus a suite of worked examples that any implementation is checked against. The formalism was surveyed against the 2026 field and confirmed (`notes/grammar-formalism.md`): PEG for the inline layer, a numbered small-step algorithm for the block layer. Two execution rules came out of that survey: the PEG lands as a machine-readable file the reference parser is generated from or tested against in CI, never hand-transcribed, and every numbered block step points at the conformance examples that exercise it.
2. **Reference parser** — the first realization, in Rust, producing a documented syntax tree.
3. **The tools that make it worth adopting** — a validator with genuinely good error messages, a canonical formatter, and a migrator from Markdown. This is where most of the value to users lives.
4. **Ecosystem** — cheat sheet, web playground, conformance badge, editor highlighting (a tree-sitter grammar — a realization tested against the suite, normative over nothing, because its notation tolerates ambiguity), and a short language description sized for an AI model's system prompt, validated by running its output through the suite rather than by proofreading it.

## Ideas parked

None of these are commitments. They are things worth remembering when the relevant rung arrives.

**Tooling that writes into the source.** The pattern behind three separate decisions: capability lives in tools, and tools write plain text into the document rather than doing anything at render time.

- A bibliography tool that reads a `.bib` file and writes citations and a reference list into the document, hanging them on anchors.
- Figure, table, and section numbering built on anchor naming conventions, using names rather than numbers so inserting one figure does not rewrite everything after it.
- A table-of-contents generator that inserts a real list of links into the source.
- A snippet-sync tool that pulls code out of a source file and writes it in, so drift shows up in the diff instead of hiding.

**Editor support.** Not part of the language; the natural home for friction relief.

- Typing help for the backtick, which is a dead key on several non-US keyboard layouts.
- A prompt for image alt text while the author still has the picture in mind.
- Inline preview of anchors and cross-references.

**Optional extras.**

- Base64 image embedding as a "shrink-wrap" step for a self-contained file. Never a requirement — inlining binary wrecks diffs and makes a light format heavy.
- A companion appendix listing conventional names — what `math` means, what `rust` means, which anchor prefixes tools have settled on. Explicitly non-binding, so it can be updated without touching the standard.

## Errands with outside dependencies

These need someone else — a registry, a registrar, a lawyer. Worth advancing whenever the opportunity appears. The master copy is `.claude/decisions.md` §10, and `CLAUDE.md`'s checklist is the third copy — update all three or they drift.

- [ ] Claim the `@markleft` package scope on npm
- [ ] Claim `markleft` (or `markleft-core`) on crates.io
- [ ] Register a domain — `markleft.org` or `markleft.ca`
- [x] Set the GitHub About box and topics on `markleft` — **done**; a description and five topics are live. The live wording differs from the proposal that follows, and the sixth topic was not applied; whether to adopt them is an editorial call, not an errand. Proposed description: *"A formally specified successor to Markdown — prose-safe by construction, exactly one parse for every input. This repository is the standard."* Proposed topics: `markdown`, `markup-language`, `commonmark`, `specification`, `formal-grammar`, `markleft`. Six on purpose — more dilutes to the same effect as none
- [ ] Give `tests` and `.github` their own About boxes, consistent with the above and distinct from each other. `tests` is the key comparison, not a copy of the standard
- [x] Add `LICENSE` (CC BY 4.0) to `tests` — **done 2026-08-09.** That repo had said "to be released CC0 pending review" since before the licence was settled on 2026-08-07, so it contradicted the record for two days. Worth remembering as a pattern: **a decision recorded in one repo does not propagate to the others by itself**, and the repos most likely to go stale are the ones nobody is editing
- [ ] Legal: trademark search
- [ ] Legal: ask counsel the *prior* question — is this Crown copyright at all? The Act turns on whether work was done under direction or control, not on who employs the author, so a personal project may not be caught. Cheaper to answer than the waiver question, and would settle it. **The licence itself is settled: CC BY 4.0.** What is left is this question, and the exact wording of the notice. Needs counsel, and does not imply anyone supervises the project

## Open questions

Mostly not blocking. These are answerable, just not answered.

- **Four of the grammar findings remain open**, listed in `.claude/grammar.md` in full: whether a block may interrupt a paragraph, which is now the largest item on that list; tilde fences, which the definition now answers as drafted (not inherited — Appendix D.5, pending confirmation); HTML entity references (Appendix D.14 — its strikethrough half closed by decision 22, which adopted `~{…}`); and whether a brace group may span a line ending (Appendix D.12, raised by decision 20).

- **Three are closed, and all three closed the same way — by moving the construct rather than the invariant.** Pipe tables needed a line of lookahead, and the table rewrite removed it. Uniform escaping was the only decision that *widened* the collision surface, and decision 20 replaced the rule instead of adding the validator warning that had been proposed. Shortcut reference links were the only construct depending on text arbitrarily far away, and decision 20 removed them. **That pattern is now three for three and is the most reusable thing on this page.**

- **Fourteen more in `definition.md` Appendix D.** Five the draft answers for the first time and needs confirmed (invalid UTF-8, leading BOM, matching, "column", and tilde fences); eight remain open, two of them — the brace group spanning a line ending, and whether `**{…}` is worth having — raised by decision 20; D.6 is closed by decision 19, its number kept; and D.14's strikethrough half is closed by decision 22, leaving entity references as the substantive remainder.

- **Should the governance sections split out?** Licensing, governance, and versioning are genuinely charter material sitting in a document that is otherwise a specification. If they split, that document gets the name `charter.md` back, correctly used this time.
- **What should the formatter normalize?** Several small choices are deferred to when the formatter exists: the order of tokens inside a decorator, whether ordered lists are written all-`1.` or numbered in sequence, and whether a redundant label is removed.
- **Which Unicode version, if any, to pin.** The encoding is frozen but Unicode is not, so any rule consulting a Unicode table can drift. Current recommendation is to define those rules so they never consult one.

## Worth writing down eventually

Things the record has *derived* rather than assumed. They explain the shape of several decisions at once, and a reader who has them can predict the rest.

- **Keep the unit of the source close to the unit of the edit.** Change one thing, and the diff should show one thing. This now decides four unrelated calls: not hard-wrapping prose, normalizing ordered-list numerals, naming anchors rather than numbering them, and block table cells — the last being the only one of the four where Markdown offers no good spelling at all, since a table row *is* a line.

- **Do not encode a position in something other things point at.** Positions shift; references do not want to. The narrower sibling of the rule above, and the reason anchors are named rather than numbered.

- **Do not encode a value that a tool will later compute better.** A hand-written number overrides every future improvement to the code that would have derived it, and it goes stale silently besides. This is why there are no column width hints, and it is the general form of the argument against hand-maintained list numerals.
- **Rarity in ordinary use is what makes a mark safe to give a job — and a rare *pair* is cheaper than a rare character.** The backtick survives as a delimiter because nobody types it in prose, and a colon works as a namespace separator for the same reason. Decision 20 is the general form: no character needs to be rare if the two-character sequence is, which is how `*`, `#`, `!`, and `\` all became safe while staying free on their own — and how `@` could be spent at all, at severity 1, in one position. The dollar sign lost its job for want of this rule fifteen years earlier.

- **Where two artifacts must agree, a machine checks the pair.** The grammar file and the reference parser, a `.markleft` source and its generated `.md` companion, the document and the suite — each pair is kept honest by CI, never by discipline. The general form of the licence-drift lesson: a decision recorded in one place does not propagate by itself, and the copy nobody is editing is the one that goes stale. Newest instance: `notes/grammar-formalism.md`, rule 1.

- **Audit the rules that read best.** Decision 3's "backslash before any character, no exception list" was the cleanest sentence in the record and was silently mangling Windows paths and regular expressions — worse than the CommonMark behaviour it diverged from. It survived two days and a finding that recorded the symptom, because nobody re-read a rule that sounded that good. A rule nobody questions is a rule nobody checks.
- **The tooling writes into the source, never into the render.** One policy, arrived at separately for tables of contents, file includes, and citations.
- **The parser is total and local; the validator carries every judgement.** Unknown names, duplicate anchors, dangling fragments, missing alt text, structural tabs, redundant labels, a fence closed only by the end of its container, an empty superscript or subscript group, a decorator after a non-backtick, a sigil before a bare alphanumeric run — ten of these now, consolidated in `definition.md` §4.2. The parser never fails on them; the validator warns.

## How to use this file

Add to it freely and in whatever words come. Prune when something ships or is decided. If an entry starts needing a paragraph of justification, that is the signal it belongs in the decision record instead — write it there and leave a one-line pointer here.
