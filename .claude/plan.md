# Plan

Where the project is, what comes next, and everything parked along the way. Plain language on purpose — nothing here needs a spec background to read.

**What this file is not.** It is not the decision record, which is `.claude/decisions.md` and holds *why* things were decided. It is not the rule list either, which is `.claude/grammar.md` and holds *what each construct takes away from plain text* — check a new decision against that file before recording it. It is not normative, not part of the released standard, and nothing here binds anything. When an idea here turns into a decision, it moves to the decision record and comes off this list. Keep it scruffy and current rather than tidy and stale.

## Right now — Phase 0

Two things stand between us and a stable language. Everything else is downstream.

- **Settle the definition.** `definition.md` is now a full draft — twelve sections and four appendices, the language stated in prose in §4 and the seventeen decisions with rationale in §5. What is left is the author's read-through and Appendix D, which lists every point the draft leaves open. Four of those it answers for the first time and needs confirmed; six it does not answer at all. Confirming any of them is a decision, so it goes in the record before it goes in the prose.

- **Write the prose corpus** (in the `tests` repo, `corpus/`). Original prose written specifically to *break* prose-safety — money amounts, `snake_case`, `5 * 3 = 15`, shell snippets, file globs, lines that accidentally start with `#` or `-`. Deliberately deferred until the definition settles, so the corpus is written against a stable target. Everything in it must be written by contributors; nothing scraped or borrowed.

The corpus matters more than it looks: it is the only thing that produces *evidence* rather than reasoning. Three decisions carry real risk of annoying people who already know Markdown — headings, code blocks, and lists — and nothing has tested them against actual prose.

## What comes after

The ladder, in order. Nothing gets built ahead of its rung; the definition fixes the language before any parser encodes it.

1. **Grammar and tests** — the formal grammar, plus a suite of worked examples that any implementation must pass.
2. **Reference parser** — the first realization, in Rust, producing a documented syntax tree.
3. **The tools that make it worth adopting** — a validator with genuinely good error messages, a canonical formatter, and a migrator from Markdown. This is where most of the value to users lives.
4. **Ecosystem** — cheat sheet, web playground, conformance badge, and a short language description sized for an AI model's system prompt.

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

These need someone else — a registry, a registrar, a lawyer. Worth advancing whenever the opportunity appears. The master copy is `.claude/decisions.md` §10; update both or they drift.

- [ ] Claim the `@markleft` package scope on npm
- [ ] Claim `markleft` (or `markleft-core`) on crates.io
- [ ] Register a domain — `markleft.org` or `markleft.ca`
- [ ] Set the GitHub About box and topics on `markleft` (needs the web UI or `gh`). Proposed description: *"A formally specified successor to Markdown — prose-safe by construction, exactly one parse for every input. This repository is the standard."* Proposed topics: `markdown`, `markup-language`, `commonmark`, `specification`, `formal-grammar`, `markleft`. Six on purpose — more dilutes to the same effect as none
- [ ] Give `tests` and `.github` their own About boxes, consistent with the above and distinct from each other. `tests` is the key comparison, not a copy of the standard
- [ ] Add `LICENSE` (CC BY 4.0) to `tests`, which matters more than here since that repo is designed to be vendored wholesale
- [ ] Legal: trademark search
- [ ] Legal: ask counsel the *prior* question — is this Crown copyright at all? The Act turns on whether work was done under direction or control, not on who employs the author, so a personal project may not be caught. Cheaper to answer than the waiver question, and would settle it. **The licence itself is settled: CC BY 4.0.** What is left is this question, and the exact wording of the notice. Needs counsel, and does not imply anyone supervises the project

## Open questions

Mostly not blocking. These are answerable, just not answered.

- **Five questions the first pass over `.claude/grammar.md` turned up**, listed there in full: whether a block may interrupt a paragraph (now worth more than it was, since it would drop the table rule's collision back down); a validator warning to soften the one decision that widens the collision surface; whether tilde fences exist; whether strikethrough and HTML entity references are absent by decision or by accident; and whether shortcut reference links should go, being the only construct in the language that depends on text arbitrarily far away.

- **The sixth is closed.** Pipe tables needed a line of lookahead, which was the one thing blocking Phase 1. The table rewrite of 2026-08-08 removed it by changing the construct rather than the invariant — worth remembering as a pattern: when a finding points at an invariant, try moving the construct first.

- **Ten more the first full draft of `definition.md` turned up**, listed in its Appendix D. Four the draft answers for the first time and needs confirmed; six it does not answer.

- **Should the governance sections split out?** Licensing, governance, and versioning are genuinely charter material sitting in a document that is otherwise a specification. If they split, that document gets the name `charter.md` back, correctly used this time.
- **What should the formatter normalize?** Several small choices are deferred to when the formatter exists: the order of tokens inside a decorator, whether ordered lists are written all-`1.` or numbered in sequence, and whether a redundant label is removed.
- **Which Unicode version, if any, to pin.** The encoding is frozen but Unicode is not, so any rule consulting a Unicode table can drift. Current recommendation is to define those rules so they never consult one.

## Worth writing down eventually

Things the record has *derived* rather than assumed. They explain the shape of several decisions at once, and a reader who has them can predict the rest.

- **Keep the unit of the source close to the unit of the edit.** Change one thing, and the diff should show one thing. This now decides four unrelated calls: not hard-wrapping prose, normalizing ordered-list numerals, naming anchors rather than numbering them, and block table cells — the last being the only one of the four where Markdown offers no good spelling at all, since a table row *is* a line.

- **Do not encode a position in something other things point at.** Positions shift; references do not want to. The narrower sibling of the rule above, and the reason anchors are named rather than numbered.

- **Do not encode a value that a tool will later compute better.** A hand-written number overrides every future improvement to the code that would have derived it, and it goes stale silently besides. This is why there are no column width hints, and it is the general form of the argument against hand-maintained list numerals.
- **Rarity in ordinary use is what makes a mark safe to give a job.** The backtick survives as a delimiter because nobody types it in prose, and a colon works as a namespace separator for the same reason. It is also why the dollar sign had to lose its job.
- **The tooling writes into the source, never into the render.** One policy, arrived at separately for tables of contents, file includes, and citations.
- **The parser is total and local; the validator carries every judgement.** Unknown names, duplicate anchors, missing alt text, structural tabs, redundant labels — six of these now. The parser never fails on them; the validator warns.

## How to use this file

Add to it freely and in whatever words come. Prune when something ships or is decided. If an entry starts needing a paragraph of justification, that is the signal it belongs in the decision record instead — write it there and leave a one-line pointer here.
