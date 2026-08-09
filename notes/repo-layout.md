# Memo: repository layout for the markleft-lang organization

*Non-normative. Status: decided 2026-08-06, revised 2026-08-07 once the normative hierarchy was settled. Records the organization and repository structure chosen at Phase 0, and the reasoning behind each split, so that later sessions find the analysis instead of rebuilding it.*

## Decision

The GitHub organization `markleft-lang` is the top-level home for every project repository, and the layout mirrors the normative hierarchy in `notes/normative-hierarchy.md` — one repository per layer.

- `markleft-lang/markleft` — **the standard itself**: definition, grammar, normative prose, deltas appendix. CC0-intent.

- `markleft-lang/tests` — the conformance suite and prose corpus: the key comparison. Standalone and vendorable. CC0-intent.

- `markleft-lang/markleft-rs` — the Rust realization: reference parser, canonical formatter, migrator, playground. MIT. Deferred to Phase 2; it does not exist yet.

- `markleft-lang/.github` — organization profile and shared templates.

The namesake repository holds the definition rather than the code. That is deliberate: it is the strongest available statement that the document is normative and the implementation is not, and it makes the point in the org listing before anyone reads a word of the definition. Cloning `markleft` gets you the language.

Every other repository is deferred with an explicit trigger, listed below. Nothing is split out for tidiness; a split needs a reason that a directory cannot satisfy.

**Revision note.** The original version of this memo put the specification inside a working monorepo alongside the parser and tooling. That predated the decision that the document, not the suite, is normative. Once the hierarchy was settled the monorepo stopped making sense: it filed the definition as one directory among several inside a repository named after its own implementation, and it left `markleft` mixed-licence. Nothing had to move, because every implementation directory was still hypothetical.

## Why an organization rather than a personal account

A repository under an individual account signals a personal project, and a standard needs to look like something others can rely on and eventually help maintain. Three further reasons make the choice one-way rather than merely preferable:

1. **Succession.** The language should be able to outlive the people who wrote it. An organization can add and remove maintainers without any transfer of ownership; a personal account cannot.

2. **Migration cost.** Transferring a repository between accounts later rewrites every clone URL, every citation, and every vendored submodule reference. Organization renames, by contrast, leave redirects in place — which is also what keeps the bare `markleft` name recoverable if the trademark process ever frees it (see `.claude/decisions.md` §8).

3. **Precedent.** `rust-lang/rust` is the shape readers already recognize for a language project meant to outlive its first maintainer.

## Repository: markleft — the standard

```
markleft/
  README.md      # what this is; states the normative boundary
  definition.md     # normative
  grammar/       # normative
  deltas.md      # normative — the translation guide
  notes/         # non-normative memos, like this one
  CLAUDE.md      # project scaffolding — not part of the standard
  .claude/       # decision record and prior-art survey — likewise
```

**The standard sits at the repository root. There is no `spec/` directory.** An earlier version of this memo kept one, on the argument that it put the licensing boundary on a directory line — everything under `spec/` normative, everything outside it scaffolding. That argument does not survive contact with the decision it sits inside. If the repository *is* the definition, then nesting the definition one level down reintroduces exactly the framing the layout decision removed: the standard as one component among several. It also puts `spec/` in every URL and every citation, for a distinction that a single sentence can make instead.

The boundary is therefore drawn by exclusion: **everything in this repository is the standard except `CLAUDE.md` and `.claude/`**, which are working scaffolding — neither normative nor part of what gets released. `README.md` says so, so a reader or a lawyer can settle it without opening anything else. The exclusion list is short and stable, which is what makes this cheaper than a directory.

The standard contains the standard and nothing else: no tests, no measurement data, no tooling, no implementation notes. The sole exception is `notes/`, which holds non-normative memos like this one, kept adjacent because they record why the normative text says what it says. A reader can read the standard without reading the project.

## Repository: tests

The conformance suite is its own repository rather than a directory, because its audience is not this project. Third-party implementations in other languages need to be able to vendor it — as a submodule, a subtree, or a release tarball — without cloning a Rust parser, a website, and a migrator they will never build. That is the whole point of a suite meant to serve as the key comparison: it has to be cheap for a stranger to run against their own parser.

It is also the cleanest possible licensing surface. The repository is CC0-intent in its entirety, with no MIT code anywhere inside it, so a vendoring implementer never has to reason about which directory carries which terms.

```
tests/
  examples/      # spec.txt-style paired examples — input, expected output
  corpus/        # prose documents authored for this project (see below)
```

**Consequence for the working convention.** Under CLAUDE.md a grammar or spec change lands together with its executable examples in the same commit. Across two repositories that is no longer literally possible. The rule becomes:

- A spec change and its examples land as a **paired pull request**, opened together and merged together; neither merges alone.

- The `tests` repository records the spec revision it targets, so any checkout of the suite states unambiguously which version of the standard it is testing.

- The spec is never tagged for release ahead of the suite that exercises it.

This is weaker than atomicity and it will occasionally be violated by accident. The mitigation is CI in the `markleft` repository that fails when normative text changes without a corresponding referenced revision bump in `tests` — worth building early, because the failure mode (a standard whose suite silently lags it) is exactly the drift this project exists to prevent.

Note that the pairing burden is not a cost of splitting the specification out of a monorepo. It exists between `markleft` and `tests` regardless of where the specification lives, because those are the two artifacts that must move together. Moving the specification did not add a boundary; it relocated one. Coordination with `markleft-rs` is a different matter and needs no pairing at all: an implementation follows the standard, it does not co-evolve with it.

## Corpus: original material only

`tests/corpus/` holds prose documents **written for this project**. Nothing is scraped, imported, quoted, or adapted from anywhere else — no READMEs collected from GitHub, no excerpts from published writing, no third-party samples of any kind. Every file is original work by a project contributor, released under the same CC0-intent terms as the rest of the repository, so the suite carries exactly one licence and a vendoring implementer inherits no obligations from it.

This directly serves invariant 1: `.claude/decisions.md` §3 specifies that prose-safety is "testable via a prose corpus in CI," and that is what this corpus is for. Documents should be written to *attack* the invariant, not to flatter it — money amounts and bare dollars, `snake_case` identifiers, arithmetic like `5 * 3 = 15`, shell snippets, file globs, emoticons, mid-word punctuation, and prose that happens to begin a line with a hash or a hyphen. A corpus of well-behaved paragraphs proves nothing.

**The known weakness, stated plainly.** We author the documents, so we author documents our rules already handle. Selection bias is unavoidable here in a way it would not be with collected material, and no amount of adversarial intent fully removes it — you cannot reliably write the input you did not think of. Two partial mitigations are worth the effort: recruit prose from contributors who have not read the grammar, and treat every real document that breaks under the Phase 3 migrator as a corpus submission.

What this corpus explicitly does **not** provide is the "percentage of real documents unchanged" figure. That metric required real documents, and it was to be the empirical check on decisions 4, 5, and 6 (ATX-only headings, fenced-only code, strict lists) — the three the decision record flags as carrying the most muscle-memory risk. Expect design challenges to those three to arrive during Phase 3, from the migrator's change report, rather than during Phase 0 when they would have been cheap to act on.

## Governance: definition prose, not a repository

There is no governance repository, and there should not be one yet. A governance repository needs a *process* to house, and the process today is that one maintainer decides and writes the decision down. What exists instead:

- **Governance posture** — who maintains the language, licensing intent, the reserved conformance mark, and the explicit statement that no institution governs or endorses the project — is a section of `definition.md`. It is a position people will cite, so it belongs in the document meant to be cited.

- **Contributor-facing process** — a contributing guide, a code of conduct — goes in the `.github` repository when the first outside contributor appears. That repository exists already, so this costs nothing to adopt later.

A dedicated repository earns its keep only alongside the RFC process, and splits out on the same trigger: more than one decision-maker. Standing one up sooner produces an empty repository full of ceremony and no contributors, which is precisely the failure `.claude/landscape.md` records against AsciiDoc — complexity front-loaded, while adoption is decided in the first five minutes.

## Where CLAUDE.md lives

Each repository carries its own `CLAUDE.md`, and Claude Code sessions are launched from inside a repository rather than from the organization folder.

The organization folder is not a repository — it is an ordinary directory holding three of them — so a `CLAUDE.md` placed there could never be committed, reviewed, or cloned. It would exist on exactly one machine and vanish when anyone else checked the project out. Claude Code *would* load it, since it walks up from the working directory, which is what makes the trap an easy one to fall into.

The satellite files stay deliberately thin. `tests/CLAUDE.md` and `.github/CLAUDE.md` state only what is needed to work in those repositories and point at the canonical sources — the invariants and decisions in `markleft/CLAUDE.md`, the record in `markleft/.claude/decisions.md`. Restating normative content in three files would give it three chances to drift, which is the same failure this memo exists to prevent elsewhere.

## Deferred repositories and their triggers

- **`rfcs`** — created when the project has more than one decision-maker. The migration is natural: the binding decisions become RFC-0001 onward, retroactively.

- **`tree-sitter-markleft`** — the tree-sitter ecosystem fixes both the repository name and the standalone-repository layout; this one cannot live in a subdirectory. Phase 4.

- **`vscode-markleft`** and any later editor plugins — marketplace publishing needs a release cadence and version numbers independent of the spec version. Phase 4.

- **`markleft-rs`** — the Rust realization. Created at Phase 2, when the parser starts. Carries the formatter, the migrator, and the playground with it, since all three are products of that realization rather than of the language.

- **Further independent implementations** — `markleft-py`, `markleft-cpp`, and so on. Created when someone writes one, by whoever writes it.

## Naming conventions

Repositories do not repeat the organization name: `markleft-lang/markleft`, never `markleft-lang/markleft-lang`. The one exception is `tree-sitter-markleft`, whose name is fixed by an external ecosystem.

**`markleft-<lang>` is reserved for independent implementations**, one per language, each written against the specification. The pattern is doing real work: it says in the org listing that realizations are plural and interchangeable, which is the whole claim of the normative hierarchy.

**A binding is not a realization, and does not take the name.** A thin wrapper over the Rust core — via WASM, FFI, or a subprocess — belongs in `markleft-rs/bindings/`, because it is a way of reaching that realization rather than an independent one. The distinction is not pedantry: running the conformance suite against a wrapper compares a realization with itself, which measures nothing while looking exactly like a passing key comparison. An ecosystem of five bindings and one parser has *one* implementation, and a standard that cannot tell the difference has lost the guarantee the suite exists to provide.

`markleft-lang/tests` reads unambiguously in the organization listing. It reads less well once vendored, where a checkout named `tests` inside someone else's project says nothing about whose tests they are; `conformance` would survive that context better. The rename is free today and expensive after the first implementation vendors the URL, so it is worth settling before the repository is created rather than after.

## A note on this file's extension

This memo dogfoods the language rules — ATX headings, fenced code only, `*` emphasis only, no underscore emphasis — but carries `.md`, because no Markleft renderer exists yet and GitHub must be able to display it. Rename the spec documents to `.markleft` once the reference parser and site rendering land in Phase 2.
