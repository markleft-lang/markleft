# Working conventions

Project-specific rules for Claude Code sessions in the `markleft-lang` repositories. The two portable modules (`markdown-conventions.md`, `git-conventions.md`) contain no project content by design; this file holds what they cannot. It says how to work — facts about the language belong in the decision record, and nothing here restates them.

## Decisions and the record

- `.claude/decisions.md` is the record. Consult it before revisiting anything settled there; if new evidence (corpus failures, migrator change reports, implementer bug reports) challenges a decision, don't silently change it — write the challenge into a `notes/` memo and flag it.

- Every contribution holds the five invariants (§3). A proposal that breaks one cannot land as it stands — changing that means amending the invariant itself, in the open, with the argument written down.

- Cite decisions by number ("challenge to decision 6") — in prose, in commits, and in `notes/` memos.

- Before recording a decision, check it against `.claude/grammar.md` — the reserved-position table first, then the rules — and add the new rule and any table change in the same session, in the same edit.

- Amendments are appended to the record, never edited in silently. Anything confirmed out of `definition.md` Appendix D goes into the record as an amendment, not silently into the prose.

- The open-checklist master is decisions §10, mirrored in `.claude/plan.md` — update both when an item closes, or they drift.

- Don't build ahead of the phase (roadmap: decisions §9) — the definition fixes the language before any parser encodes it. When `markleft-rs` lands, add a Commands section to `CLAUDE.md` that covers running a *single* conformance example, because that is the unit of debugging.

- Tags are waypoints, not per-commit; the version bands (`0.x.0` language change, `0.x.y` everything else, `1.0.0` normative-and-passing) are in decisions §11 and `definition.md` §12. Merge exploratory work with `--no-ff` so the bubble stays visible and droppable.

## Normative hierarchy and cross-repo workflow

- The document is normative; the suite is the key comparison — mandatory but subordinate. When they disagree, the document governs and the test is presumed to be the bug, unless the test is evidence the spec needs revising — a spec change through the decision record, never a quiet edit. Full triage: `notes/normative-hierarchy.md`.

- A spec change and its conformance examples land as **paired pull requests** across `markleft` and `tests` — opened together, merged together, neither alone. `tests` records the spec revision it targets; the spec is never tagged ahead of the suite that exercises it. See `notes/repo-layout.md`. (A workflow rule of this project, which is why it is not in the portable git module.)

- The public status board is the phase checklist in the `.github` repo at `profile/README.md` — the single source of truth. Check a phase off when it completes, and add or remove phases there in the same edit that changes decisions §9; this repo's `README.md` links to it and must never grow a second copy.

- `tests/corpus/` is contributor-authored original material only — written to attack invariant 1, never scraped, imported, quoted, or adapted. Full rule: `notes/repo-layout.md`.

## Prose and voice

- Watch the voice: authoritative language (*forbids*, *rejected*, *required*) earns its place when an objective ruling is doing the work — a grammar rule, a program, a definition — and not when a person is. Test and substitution corpus: `notes/voice-and-authority.md`.

- Prefer boring, explicit rules over clever heuristics — cleverness is how Markdown got here.

- Humor budget: the "What's left?" pun and the extension graveyard are canon; everything else in normative text stays sober.

- Attribute inheritances explicitly in spec prose — CommonMark, djot, MyST/rST, Markdoc, Typst, AsciiDoc/Eclipse; what each contributed is in `.claude/landscape.md`. Most of our syntax is djot-vetted; saying so is both accurate and disarming.

## Markdown, not Markleft (do not dogfood)

- Project `.md` files are Markdown and are not written in Markleft. Decision 20's clean break means a document containing a link is one language or the other, and GitHub renders only Markdown — so write ordinary CommonMark (`[text](url)`, `*em*`, `![alt](src)`). Writing Markleft syntax into a `.md` file produces literal punctuation, which is worse than not dogfooding.

- **Escape dollars in `.md` files** — GitHub's math pass pairs them on render, which is the founding exhibit happening to us.

- When a document quotes Markleft syntax, put it in a fence or a verbatim span.

- Dogfooding is separate from source formatting: the Markdown conventions module governs how every `.md` file here is formatted, this one included; dogfooding governs which constructs are permitted, and is settled above.

- Self-hosting arrives in one stage at Phase 3 — `.markleft` sources with generated `.md` companions, kept in sync by CI. Until then, `.md` files are authored by hand. Detail: decisions §9.

## Licensing notices (apply to every new file)

The licensing substance and rationale are decisions §7; these are the drafting rules for the files themselves.

- Spec and suite files: CC BY 4.0 — mark them "Crown copyright, Canada — 2026. Licensed CC BY 4.0." All code: MIT.

- Crown copyright attaches by operation of law, not by choice, and is not sponsorship. Never phrase a notice so a reader could infer a department backs, owns, supervises, or vouches for the project.

- Never write the monarch's style into a notice, and never write "Government of Canada" either — "Crown copyright" is the term of art, names no person and no institution, and is what a reuser searches for.

- Public files state the position; they never show the reasoning or hedge it. The wording is provisional pending counsel — that fact lives in the decision record, not in a notice.

- Treat every notice string in these repos as a placeholder: exact wording is a question for counsel, not something to invent here.

- Do not claim OGL-Canada declares CC BY compatibility; it does not — that clause is the UK OGL's.

## Names that are settled

- File extensions: `.markleft` and `.lf`, exactly two. Never use anything from the graveyard in decisions §6 (`.lft` is reserve fallback only, not a live option).

- Repositories live under the `markleft-lang` org, never a personal account. `markleft-<lang>` is reserved for independent implementations; a binding is not a realization and belongs in `markleft-rs/bindings/`. Full rationale and naming rules: `notes/repo-layout.md`.
