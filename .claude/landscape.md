# Markleft — Prior Art Landscape

*Survey of projects that have tried to "fix," extend, or replace Markdown. Researched August 2026. Companion to `.claude/decisions.md`; feeds the charter's prior-art section. Organizing lens: the size/formality quadrant map — Markleft targets **small language + formal spec + prose-safety**, a quadrant no project below occupies.*

---

## Strategy 1 — Standardize Markdown as it is

### CommonMark (2014, John MacFarlane et al.)

The heroic effort: a rigorous prose spec + ~650 executable examples that became the de facto baseline (GFM is formally a superset of it). What it proved: spec-as-tests works; a reference implementation + conformance suite can align an ecosystem. What it couldn't fix: the syntax itself — emphasis needs 17 interlocking rules + a delimiter stack (not context-free), list indentation archaeology, seven kinds of HTML block, no math, no extension mechanism ever shipped. MacFarlane's own post-mortem ("Beyond Markdown") concedes the unfixable parts. **Lesson: standardizing the accident preserves the accident. Our inheritance: the spec-as-tests methodology.**

### GFM spec (GitHub)

CommonMark + tables, strikethrough, task lists, autolinks. Math is NOT in the spec — it's platform post-processing (MathJax, added 2022), the architecture that garbles dollars. **Lesson: bolted-on layers outside the parser are the enemy; the founding exhibit of this project.**

## Strategy 2 — Replace with a cleaner small language (our nearest neighbors)

### djot (2022, John MacFarlane) — closest prior art

Direct implementation of the "Beyond Markdown" ideas: linear-time parsing, no backtracking, uniform escaping, attributes on any element, generic containers, math, footnotes, tables, definition lists. Design principles we share: no expressive blind spots; simple list-item rules; parsers shouldn't need Unicode classes/HTML tags/entities; block elements can't interrupt paragraphs. Status (2026): syntax still "not completely stable"; the JS implementation (djot.js) is the complete reference; original Lua may lag; community ports in Rust, Go, PHP, Prolog, Haskell (the Haskell parser is used by Pandoc); started in a personal repo (jgm/djot), later grew an org. No formal grammar; spec is prose + reference code. Kept `_` emphasis and smart punctuation; math/money not a headline concern; no validator/formatter/migrator tooling story. **Markleft delta: formalization (grammar + normative suite + canonical AST), prose-safety as invariant #1, dollar verdict, killer tools. Everything djot got right, we adopt with attribution.**

### Quarkdown (v2.0 April 2026, Kotlin/JVM)

"Markdown with superpowers": single source → paper/slides/site/notes via a `.doctype` directive; templating; standard library. In practice a typesetting language with markdown-shaped syntax (family: LaTeX/Typst/AsciiDoc). No compatibility mode with existing Markdown pipelines — adoption is a switch, not incremental; commentators note it thereby inherits AsciiDoc's exact adoption problem. **Lesson: "markdown-shaped" is not "markdown-compatible"; our byte-for-byte-where-harmless principle is the answer.**

## Strategy 3 — Extend Markdown upward for scientific/structured publishing

### MyST — Markedly Structured Text (Project Jupyter, MIT license)

Superset of CommonMark adding Sphinx-inspired **directives** (block extension points) and **roles** (inline extension points), citations, cross-references, math. Drives Jupyter Book 2; JS implementation on the unified ecosystem; community-governed. This is the most principled *extension mechanism* in Markdown-land — conceptually near our closed extension namespace, but grown as a superset of an ambiguous base, so it inherits every CommonMark pathology underneath the directives. **Lesson: a clean extension story on a dirty core is a penthouse on a swamp. Also: MyST↔Quarto conversion requires a dedicated third-party tool — same-family dialects can't even interconvert trivially.**

### Quarto (Posit PBC, Pandoc-based, GPL)

R Markdown's successor: `.qmd` = Markdown + executable code cells + YAML config → HTML/PDF/DOCX/JATS. Corporate-driven (Posit) vs MyST's community governance — the ecosystem now has two competing scientific-markdown stacks with incompatible directive syntaxes. Quarto is a *publishing system*, not a language spec; its markup is "whatever Pandoc accepts" plus conventions. **Lesson: toolchain gravity beats language design — people adopt Quarto for outputs, not syntax. Markleft is a language, not a toolchain; keep it so.**

## Strategy 4 — Extend Markdown for docs-as-data / components

### Markdoc (Stripe, 2022, MIT)

Declarative `{% tag %}` syntax over Markdown with **schema-validated** attributes; strict code/content separation ("docs as data"); AST + static analysis + validation; `.mdoc`; LSP work. Explicitly inspired by AsciiDoc. The closest anyone has come to "Markdown with a validator" — but validation covers the tag layer, not the Markdown substrate, which remains ambiguous markdown-it underneath. **Lesson: there is real demand for validation and machine-readable markup; nobody has offered it for the base language itself. That's our Phase 3.**

### MDX

Markdown + embedded JSX; "docs as code"; arbitrarily complex JavaScript in content; large runtime footprint; content becomes code with code's maintenance burden. Huge React-world adoption. **Lesson: the anti-pattern we define ourselves against — maximal power, zero prose-safety, unparseable without a JS engine. Also proof that platform integration drives adoption regardless of language virtue.**

## Strategy 5 — Markdown-flavored typesetting (the heavy-structure space)

### Typst (Rust, company-backed, stable language past v0.13)

LaTeX's modern replacement: markdown-ish markup + real scripting language + incremental compiler + excellent error messages; instant preview; growing academic use, though journals still demand .tex. Not a Markdown fix — a document *preparation* system where markup is one layer of a programming language. Notable for us: Typst chose `$...$` for math and `_` for emphasis — the very collisions we're eliminating — acceptable there because Typst documents are programs, not prose-safe plain text. **Lessons: (a) meaningful error messages are a killer feature people rave about — validates Phase 3; (b) Rust + fast compiler + playground = the modern credibility stack; (c) different design goals legitimately produce different dollar verdicts — scope discipline matters.**

### Quarto/R Markdown, LaTeX itself, etc.

See the strategies 3/5 overlap; all confirm: page-aware, programmable documents are a *different product* from a lightweight prose format. Markleft must never drift here (charter non-goal).

## Strategy 6 — The elder statesmen (pre-Markdown or parallel lineages)

### AsciiDoc (2002; Eclipse Foundation stewardship)

Semantically rich, book-class features (includes, conditionals, row-span tables, admonitions, cross-references). The spec effort under the Eclipse AsciiDoc Working Group — specification + TCK, covering syntax, an Abstract Semantic Graph, DOM, and APIs — has been "underway" since ~2020 and remains in development in 2026, with Asciidoctor still effectively the sole implementation. Full analysis of its adoption stall is in `.claude/decisions.md` §2. **Lesson: even with Eclipse-grade governance, formalizing a large language retroactively takes half a decade and counting. Formalize small, formalize first.**

### reStructuredText (Python/docutils, ~2002)

"A proper specification" before it was cool; directives/roles that inspired MyST; powered Sphinx and Python docs for two decades. Verbose, rigid, Python-bound tooling; steadily losing ground to Markdown even inside Python (Sphinx now supports MyST precisely to offer .md). **Lesson: correctness without ergonomics loses; the five-minute property is not negotiable.**

### Org-mode (Emacs, 2003)

Arguably the most capable plain-text format ever (outlining, literate programming, agendas, tables with formulas) and permanently trapped in its editor. **Lesson: editor-coupled formats don't propagate; Markleft must be editor-agnostic with LSP-grade tooling.**

## The gap Markleft targets (positioning summary)

| Project    | Size  | Formal spec        | Prose-safe            | Math story             | Validator | Steward         |
|------------|-------|--------------------|-----------------------|------------------------|-----------|-----------------|
| CommonMark | small | tests, no grammar  | no ($ ok, _ not)      | out of scope           | no        | volunteers      |
| djot       | small | prose + ref impls  | partial (_ kept)      | yes, $ delimiters      | no        | personal→org    |
| MyST       | large | superset def       | inherits CommonMark   | yes                    | partial   | Jupyter         |
| Quarto     | large | none (toolchain)   | inherits Pandoc       | yes                    | no        | Posit (corp)    |
| Markdoc    | mid   | tag schema only    | inherits markdown-it  | no                     | tag layer | Stripe (corp)   |
| MDX        | large | none               | no                    | no                     | no        | community       |
| Typst      | large | docs + impl        | n/a (it's code)       | yes ($ delimiters)     | compiler  | company         |
| AsciiDoc   | large | in progress (yrs)  | no                    | via stem blocks        | TCK (wip) | Eclipse         |
| **Markleft** | **small** | **grammar + normative suite** | **invariant #1** | **core, collision-free** | **Phase 3 centerpiece** | **independent** |

Nobody combines: small + formally specified + prose-safe + collision-free math + validation tooling — with no vendor's product roadmap attached. Every cell of that combination exists somewhere in the table; the conjunction exists nowhere. That conjunction is the project.

## Standing on shoulders — explicit inheritances to acknowledge in the charter

- **CommonMark:** spec-as-tests methodology; byte-compatibility baseline.

- **djot:** linear-time architecture, uniform escaping, attributes (as decorators), bracketed emphasis; most of our syntax decisions are djot-vetted. **Not** its raw-content design — decision 7 removes passthrough entirely, because raw HTML is transclusion and decision 15 forbids it.

- **MyST/rST:** the directive/role concept, disciplined into a closed namespace — the extension *point*, never the content-generating directive, which decision 15 forbids outright.

- **Markdoc:** proof of demand for schema validation and docs-as-data.

- **Typst:** the error-message bar, the Rust+WASM+playground stack.

- **AsciiDoc/Eclipse:** the TCK concept (= our conformance suite as key comparison); the cautionary timeline.
