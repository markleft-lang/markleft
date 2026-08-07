# Markleft — Project Decision Record

*Origin conversation: August 2026. Status: pre-charter. This document records the decisions, rationale, and open items from the founding discussion.*

---

## 1. Origin story

The project began with an investigation of the `$` sign in Markdown: no Markdown specification (Gruber 2004, CommonMark 0.31.2, the GFM spec) defines `$` as math syntax. Dollar-delimited math is entirely a renderer-level extension (GitHub added MathJax post-processing in 2022; Pandoc's `tex_math_dollars` has the most rigorous heuristics; KaTeX ships with single-dollar delimiters disabled by default).

Mid-conversation, a paragraph *explaining* this problem was itself garbled by the chat interface's bolted-on math pass — dollars paired across the paragraph, whitespace collapsed by TeX math mode, and code spans offered no protection because the math layer ran outside the Markdown parser. This live failure is the project's founding exhibit: **same source, different realizations — a traceability failure, in metrological terms.**

## 2. Mission

Fix Markdown's ambiguity without losing what made it win. Occupy the empty quadrant: **small language, formal spec** (Markdown is small/informal; AsciiDoc is large/semi-formal; djot is small and better but still informal-ish). Nearest prior art is djot (MacFarlane, 2022) — we take its good ideas, add formalization, and make prose-safety and math a headline concern.

### Why AsciiDoc lingered (lessons internalized)

- It solved writing, not *source readability* — Markdown source looks like email.
- Single-implementation problem for ~20 years; spec came very late.
- Lost the platform war (GitHub chose Markdown; network effects compounded).
- Complexity front-loaded; adoption curves are set by the first five minutes.

### New adoption variable

LLMs are now arguably the largest Markdown-producing population. A trivially learnable, formally validatable, math-collision-free language is a better output target for machines. This tailwind did not exist for any predecessor.

## 3. The four invariants (constitution — features violating one are rejected)

1. **Prose-safety.** Any natural-language paragraph renders verbatim unless it contains a delimiter in a structurally meaningful position; the set of such positions fits on one page. Testable via a prose corpus in CI.

2. **Five-minute property.** Complete core fits on one reference card. A rule needing an "unless" clause is a wrong rule.

3. **One-meaning property.** Every input has exactly one parse. The spec is executable (grammar + exhaustive tests), not interpretable prose.

4. **Linear-time property.** Single pass, prefix-decidable blocks, no backtracking, no pathological inputs.

## 4. The binding decisions

1. **Dollar is dead as syntax; math is decorated verbatim.** *(Inline form revised 2026-08-07; the word `math` left the specification the same day — decision 9.)* Bare `$` is always literal. Display math is a fence labelled `math`; inline math is a verbatim span decorated `{math}`. **"Math is core" means three spec-level guarantees, and only these three:** `$` is never math syntax anywhere; there is a first-class construct, at block and inline sizes alike, whose content is **verbatim** and therefore safe from reinterpretation; and its label is carried faithfully into the AST. Those three fix the founding exhibit in §1 exactly — what failed there was that `$` paired across a paragraph and that code spans did not protect the content, never that the word "math" lacked a definition. So `math` is a **conventional label, not a reserved word**: the specification guarantees the prose-safe, content-protected box and the faithful label, and what the label means is agreed outside the standard. That is a smaller claim than it sounds, and it is the whole of what a *language* can honestly promise about math — a renderer still chooses the typesetting, as it always did, but it can no longer change what the source says. **Stated plainly: the goal was never to specify a math engine, it was unequivocalness — to end the `$` mess.** Implementation of the rendering stays free, and `` `…`{math} `` is the right answer precisely because it draws that line in the syntax: the language guarantees what the source unambiguously *is*, and says nothing about what a renderer does with it. The original `\(...\)` inline form is **withdrawn**: it cannot coexist with decision 3, which yields the character after any backslash with no exception list, so `\(` is a literal `(` and those delimiters cannot exist. The deeper failure was in the content rather than the delimiters — inline TeX is backslash-dense, so `\(\frac{a}{b}\)` loses `\f` to an escape and offers no way to tell a closing `\)` from an escaped paren. **Math content has to be verbatim**, which is why the fenced form was never in trouble and why the inline form belongs in the same machinery: a verbatim span carries TeX through untouched. Decorator grammar: decision 9.

2. **Underscore is dead as syntax.** Emphasis is `*em*` / `**strong**` only. Bracketed form (djot-style `{*...*}`) for intra-word cases. Flanking rules collapse to two lines.

3. **Uniform escaping.** Backslash before *any* character yields that character.

4. **ATX headings only, in one closed form.** *(Riders decided 2026-08-07.)* A heading is a run of one to six `#` in the first column, then one space, then non-empty text — and **a line that does not match that form is text.** The whole rule is one sentence with a uniform fallback; every case that follows falls out of it rather than being legislated separately. **`#` is the only heading marker:** setext underlines (`===`, `---`) are removed, nothing else introduces a heading, and nothing ever will — setext also costs lookahead, since a line's block type must be decidable from its own prefix (invariant 4) and an underlined heading is a heading only because of the *next* line. **No closing sequence:** CommonMark's optional trailing run is removed, so in `## Title ##` the heading text is `Title ##`; two spellings of one heading is one too many, and the rule separating a closing run from ordinary text is exactly the "unless" clause invariant 2 forbids (`# foo #` closes, `# foo #bar` does not). **No leading indentation:** the `#` sits in the first column, after any container prefix — CommonMark's up-to-three-space allowance existed to keep headings clear of indented code, and decision 5 deleted indented code, so it now buys nothing and costs a boundary case. **No empty heading:** a lone `#`, or `#` followed only by spaces, is a paragraph whose text is `#`; decision 13 makes trailing whitespace insignificant, so `#` and `# ` are the same input and get the same answer. **Six levels, because HTML has six:** `h1`–`h6` is the ceiling of the format everything renders to, the other targets agree (DocBook `sect5`, LaTeX bottoming out at `subparagraph`), and CommonMark and GFM cap there too — so the cap is free under decision 12, where raising it would be a delta that buys nothing renderable, and a document needing a seventh level needs splitting instead. `#######` therefore fails the form and is text, which is the same fallback that keeps `#hashtag` and `#5` safe with no escape at all: neither has the required space. Only the genuinely ambiguous spelling — `# 5 is my favourite` — needs `\#`, and decision 3's uniform escape already covers it, so prose-safety here is structural rather than a carve-out.

5. **Indented code blocks are dead.** Fenced only. (Also massively simplifies list parsing.)

6. **Lists with arithmetic, not archaeology.** One marker per list; continuation aligns with content column; no lazy continuation; loose/tight by one explicit rule. *Most in need of corpus testing.*

7. **No raw passthrough. At all.** *(Reversed 2026-08-07 — previously an explicit ` ```=html ` block plus an inline raw span, djot's design.)* Markleft has no way to emit content in a target format. There is no raw block, no raw span, and no `=format` decorator; the `=` sigil leaves the language with them. **The reason is decision 15:** `<iframe>`, `<script>`, and `<object>` render content that is not in the source, which is transclusion under another name and exactly what the cardinal rule forbids. *(`<img src>` was listed here in the original argument and does not belong — an image adds no **text**, which is why decision 16 admits images. The outcome is unaffected: the three tags above are sufficient on their own, and the cannot-be-patched-by-rule argument below is independent of any list.)* Nor can the hole be patched by rule — `<br>` adds nothing and `<iframe>` adds everything, and separating them requires knowing HTML semantics, an unbounded blocklist and an "unless" clause at once, which invariant 2 refuses. **What removal buys, beyond closing the hole:** a Markleft document is **inert by construction** — it cannot execute or embed anything, ever, a claim no Markdown flavour can make and one that lands directly on the LLM-emitter argument in §2, since a safe output target is worth more when "safe" needs no qualifier; no target-format lock-in, where `=html` had put HTML *inside* a language claiming independence from any serialization, the same fault `notes/normative-hierarchy.md` finds in CommonMark binding its suite to HTML; and a simpler decorator grammar, decision 9's `=name` token disappearing entirely. **The cost, stated honestly:** an escape hatch is what lets a small language *refuse* feature requests — "Markleft cannot do X" is answerable with "drop to a raw block" only while one exists. Without it, every gap presses directly on the core, which is the force that killed the five-minute property everywhere else. `<details>`/`<summary>` is the concrete casualty on GitHub, and djot's answer to it is generic containers, which `.claude/landscape.md` records us declining. That pressure is judged better handled by saying no than by keeping a hole in the cardinal rule. **Rejected alternative:** keeping the raw block as a loudly-marked exception, where decision 15 holds *except* inside a fence that warns the reader the plain text is not the whole story. Honest, but still an "unless" clause on the cardinal rule, and the rule is worth more intact. *Open: what the migrator does with the HTML in existing documents — it cannot convert it, so it strips and reports or refuses the document. Deferred deliberately; principle first.*

8. **Pipe tables in core, strictly.** GFM-style with a real grammar; no heuristics.

9. **Decorators — one vocabulary, two positions, no reserved words.** *(Reduced repeatedly on 2026-08-07: renamed from attributes, then stripped of reserved words, then of classes.)* A **decorator list** is a space-separated sequence of tokens in exactly two shapes, **each at most one** — there is no multiplicity exception anywhere in the grammar:

    - a **bare word** — the format label. **At most one.** A single word, kebab-case permitted (`x-mermaid`, `objective-c`, `plain-text`). One is the ceiling because two format words would claim two content types for one box: there is no meaning to combine them into and no resolution order to appeal to, so a second word buys a race and an undecidable decoration rather than a capability.

    - **`#id`** — an anchor, and **navigation rather than style**. At most one, matching HTML, where an element carries a single `id`. *An id does not attach to the construct the way a word does: it marks a position — see decision 17, which is why both tokens can share one brace group while doing different jobs.* *Uniqueness across the document is a validator check, not a parse error* — a duplicate id is well-formed syntax and a broken anchor, which puts it with the lint warnings rather than in the grammar, since a document-wide constraint cannot be expressed without leaving the local, single-pass discipline invariant 4 buys.

    **There is no `.class`.** *(Removed 2026-08-07, reversing the same day's decision to keep it.)* It was the only construct in the language whose entire purpose was to let rendering vary, which is what the project refuses everywhere else in the same shape — raw HTML went for breaking uniformity of content, and classes break uniformity of presentation. Four counts against it. **It fails decision 15's own design test:** GitHub's sanitizer strips class attributes outright, so the mark is consumed and *nothing* replaces it, which is exactly the information destruction the back half forbids; the exemption granting classes a pass was convenient rather than reasoned. **It pays for nothing under the addition bar:** it removes no ambiguity and fills no gap except "Markdown has no styling", which is the richer-Markdown-for-its-own-sake trap the charter guards against. **It balloons the source:** any non-trivial CSS structure turns readable prose into a scaffold of class names, so the cost falls on invariant 1, the one that outranks everything. **And it invites mixing styling with content** — an antipattern here, and the thing many people came to Markdown to escape: being freed from styling decisions is a feature, not a deprivation. *(Author's framing, and the deciding argument.)* Adoption evidence points the same way: Markdown's lack of styling softly enforces uniformity, and that uniformity is part of why it pastes anywhere.

    Nothing else — no `key=val`, no `=format`, no colons, no sigil zoo. The id keeps its sigil because it names something that is not a format. Order within the list is free and the canonical formatter normalizes it (Phase 3). The list is written in two positions and means the same in both: **on a fence**, immediately after the opening backticks; **inline**, in braces immediately after the closing backtick run.

    ````
    ```math                `x=y`{math}                 labelled math
    ```rust                `let x = 1`{rust}           labelled verbatim
    ```math #eq1           `x=y`{math #eq1}            label plus anchor
    ```#snippet-3          `x`{#snippet-3}             anchor only
    ```                    `x`                         unlabelled verbatim
    ````

    **No word is reserved, ever — and that is the point.** The specification defines the grammar, `.class` as CSS linkage, and `#id` as an anchor. It does **not** define what any bare word means: a label is opaque, carried into the AST verbatim, and **no decorator word can affect the parse.** The grammar is therefore fixed independently of any word list — a stronger guarantee than the closed registry it replaces, because no future word can quietly become a directive when words have no parse-level power by construction, and there is no list to maintain, version, or argue about. It also removes the last place the language could have grown by accident.

    **Meanings live outside the standard.** `math` means TeX, `rust` means Rust — by convention, recorded in a **non-normative** conventional-labels appendix and in the validator's recognized-word list, both revisable without touching the specification. An unrecognized word parses as plain verbatim and earns a lint warning, never a parse error. Retaining the label in the AST is required by decision 15: a renderer may consume the mark, since highlighting and typesetting are presentation, but nothing may be lost. Vendors are advised to prefix `x-` for collision-proofing; nothing enforces it, because it is a convention rather than syntax.

    **The bare forms are defined, not defaulted.** A fence with no decorator and a span with no decorator mean **verbatim plain text, period.** That is their definition rather than a fallback, and it is the only meaning the specification assigns to a construct carrying no label. There is consequently **no `text` label** — not forbidden, because no word is ever reserved and forbidding one would reserve it, but never blessed either: the specification does not mention it, the validator flags it as redundant, and the canonical formatter removes it. Blessing it would have cost the "one way to do it" discipline for nothing, and it invites peppering prose with monospace for emphasis, which is a Markdown antipattern this language has no reason to import. *(An earlier version of this decision did bless `text` as an idempotent spelling; that was the last reserved word hiding in prose, and removing it makes the decision self-consistent.)*

    **A labelled span and an unlabelled span are different nodes.** They may render identically, but one carries a label and the other does not, and the AST faithfully records which the author wrote — exactly as decision 15 requires.

    **Legibility of the full form.** `` `x=y`{math .display #eq-euler} `` is the ceiling of the grammar rather than its normal use, and it sits at the edge of what stays comfortable in running prose — a decorator can grow longer than the content it decorates. The grammar permits it regardless, because forbidding the full form *inline only* would be an "unless" clause and the three shapes have to compose uniformly wherever they appear. House style carries that cost instead: classes and ids belong on fences, where a block has room for them, and an inline decorator normally carries the format word alone. What keeps even the ceiling inside invariant 1 is the postfix order — the content reads first, and the decoration is a suffix on a span the reader has already finished.

    *Open rider for the charter:* what character set a bare word and an id may use. Decision 14 forbids privileging an ASCII subset in constructs that take text, and a label is an identifier rather than text — the charter must say which rule governs. *Note the other brace construct:* decision 2's `{*...*}` bracketed emphasis is not a decorator. They cannot collide, since a decorator follows a backtick run and never opens with `*`, but the grammar must say so rather than leave it to inspection.

10. **Tabs are not structural.** Spaces determine structure; tabs in structural position = validator warning with auto-fix.

11. **Deltas from djot:** no smart punctuation in core; keep Markdown muscle memory where Markdown wasn't broken (links, blockquotes, backtick verbatim — challenged on keyboard-accessibility grounds 2026-08-07 and kept, `notes/backtick-verbatim-challenge.md`); formalization is our value-add over djot's prose-plus-reference-code.

12. **Compatibility principle:** match CommonMark byte-for-byte wherever it is unambiguous and harmless; diverge only to fix ambiguity or prose-harm; every delta documented in a "Deltas from Markdown" appendix doubling as the migration guide.

13. **Hard breaks are explicit; trailing whitespace is never significant.** *(Decided 2026-08-07.)* A backslash before a line ending is the only hard line break. CommonMark's "two or more trailing spaces" break is **removed**. Invisible syntax cannot satisfy prose-safety or the one-meaning property: a reader cannot see it, a diff barely shows it, editors and pre-commit hooks strip it on save to avoid whitespace noise, and the still-common habit of typing two spaces after a period turns ordinary prose into structure whenever such a sentence happens to end a line. Nobody can reasonably rely on a mark they cannot see. Same family as decision 10 — whitespace does not carry meaning. Delta from CommonMark; goes in the migration appendix, where the migrator can convert a trailing-space break into a backslash and report it.

14. **Unicode is the character set; UTF-8 is the encoding.** *(Decided 2026-08-07.)* A document is a sequence of Unicode code points encoded as UTF-8. Every code point is valid text and carries the same verbatim guarantees, as content and in every construct that takes text; no part of the grammar privileges an ASCII subset. **Content is never normalized** — normalizing would alter what the author wrote, which decision 3 and invariant 1 forbid; this clause is derived from them rather than chosen independently. *Open riders, each a potential ambiguity, to be settled in the charter:* whether a leading byte-order mark is stripped or is content; what a conforming parser does with invalid UTF-8 (reject, replace, or pass through); whether reference-label and anchor matching case-folds or normalizes, and under which algorithm; and what "column" means for decision 6's content-column alignment when a line holds wide characters or multi-code-point graphemes. That last one is the sharpest: list alignment is defined in columns, and "column" is not yet defined for text that is not ASCII.

15. **No generated content — the source is the whole document.** *(Decided 2026-08-07.)* Everything a reader will see is written in the source, in the order it appears there. A renderer may style, link, and lay out what is written; it may never supply text that is not. So there is no table-of-contents directive, no section auto-numbering, no generated index or bibliography, no file inclusion or transclusion of **text**, and no variable substitution. *(The qualifier is load-bearing: decision 16 admits images, which include a file and add no text. The line is drawn there.)* The `{.toc}`-style instruction that AsciiDoc, MyST, and Quarto all provide is declined deliberately, and its usefulness is not in dispute — that is exactly what makes it worth naming a rule rather than leaving to taste. **The cardinal rule is the test: you can always read the entire document in plain text, with no compiler, viewer, or extension.** A construct whose content exists only after rendering breaks it twice — the source stops being equivalent to the document, so a reader without the tool is reading an incomplete one; and both prose-safety and the five-minute property assume that what is on the page is what is there. This is where "not a toolchain" stops being a scoping statement and becomes a syntax rule: a generated-content directive *is* a tooling dependency written into the language, and one of them is enough to make plain-text reading conditional on having the tool. Tables of contents are genuinely useful and have two good homes, both outside the specification — the editor, which can generate one *into* the source where it becomes ordinary text like everything else (Markdown All in One and its equivalents already work this way), and the publishing pipeline downstream of the language. **Consequences:** decision 9's decorators carry metadata and may never name a construct that emits content, which closes the obvious back door; and what is inherited from MyST and reStructuredText (`.claude/landscape.md`) is the *closed extension point*, never the content-generating directive. *Open rider for the charter:* CommonMark renumbers an ordered list from its first marker, so `1.` repeated renders as 1, 2, 3 — the one inherited behaviour that generates content the source does not contain. Decision 6 has to settle whether Markleft renders list numbers as written.

    **The boundary, precisely.** The rule is not that the source must *look* like the rendered form: `**bold**` is not bold, and `` `\frac{a}{b}`{math} `` is not a typeset fraction. It is that the source must be **semantically equivalent to the rendered form, up to format** — rendering may change how written content appears, and may never create content. Emphasis markers, math notation, link text, and decorators all pass: what they mean is written down, and a reader recovers it by parsing the plain text rather than by running something, even where the typeset form is nicer to look at. A reference link passes too — its visible text is written where it appears, and only its target lives elsewhere in the same document, a target being metadata rather than content. A table of contents fails, because its entries appear where nobody wrote them. So the test for any future construct is one question: **is this changing the presentation of written content, or manufacturing content?** Presentation instructions stay admissible in principle — decision 9's decorators already are one — and how far they may go is deliberately not settled here. Content-manufacturing constructs are closed permanently.

    **The back half: rendering is lossy on marks and lossless on content.** The front half says rendering adds nothing. The converse is that rendering *removes* — `` `E=mc^2`{math} `` becomes a typeset equation and the word `math` is gone from the page. Nothing is lost by that: the statement "this is an equation" was not deleted but **re-expressed as typography**, exactly as `**bold**` surrenders its asterisks to boldness. A decorator is the plain-text stand-in for a visual distinction the plain-text reader cannot receive, which is why these are two halves of one rule rather than two rules. The constraint on this direction is narrow and absolute: **only marks may be consumed, and every unit of content survives.** A renderer that dropped the content a mark identified would break the rule as surely as one that invented content. **The design test that follows:** every format word the language defines must name something a conforming renderer visibly distinguishes. A format whose rendered output is indistinguishable from undecorated content destroys information on render, and is refused. The id is exempt by kind rather than by exception — an anchor is navigation, metadata in the same category as a reference link's target, and metadata carries no content to lose. *(Classes were exempted here too, until decision 9 removed them: a class strips to nothing on platforms that sanitize CSS, so it failed this very test.)*

16. **Images are core, and the line is textual.** *(Decided 2026-08-07.)* An image is written `![alt](src)`, **unchanged from CommonMark** — no delta, nothing to migrate. **Why an image is legal where raw HTML is not:** decision 15 governs the document's *text*, and an image contributes none. A reader of the plain source has read every word of the document. **The bend, stated honestly:** what that reader lacks — what the picture shows — is real content, and no alt text recovers it, because an image is worth a thousand words and an alt attribute is sized for a screen reader rather than for a thousand words. So the cardinal rule *is* bent here, deliberately and in one bounded place, for two reasons that do not generalize: images are essential to clarity in most real documents, and unlike text there is no way to carry one in plain text at all. A rule bent knowingly, at a named place, with the reason written down, is not the same thing as a rule with a hole in it. **Base64 embedding** would close even that gap and is worth having as an *optional* shrink-wrapping step, never a requirement — inlining binary wrecks diffs, makes a lightweight format heavy, and would discourage the very image use this decision exists to permit. An iframe imports arbitrary text that becomes part of what the reader reads, so that reader has *not* read the document — and an image is bounded and inert besides, occupying one non-textual slot with no scripts and no recursive embedding. That distinction is why "no transclusion" in decision 15 means no *textual* transclusion. **Alt text is optional and empty alt is legal.** Requiring it would enforce accessibility through the grammar, which is not the grammar's job and which invariant 2 would charge for — and the legality of an image never rested on alt text, only on its adding no text. Missing alt is a **lint warning** (screen readers will announce the filename), never an error: the sixth judgment routed to the validator rather than the parser. The path is visible in the source regardless, so a plain-text reader still learns that a picture is referenced and what it is called. **The rule that makes `!` memorable rather than cryptic:** `[…]` links to the target; `![…]` shows it instead. The `!` is a presentation switch, not a different construct — which is the same sentence that justifies an image under decision 15's back half, where alt text is content and the rendered picture is a richer presentation of it. A renderer **may** substitute the path when alt is empty; the specification does not say so, because it does not specify rendering at all (decision 1), and that guidance belongs with the non-normative conventional labels.

    **Rejected — `` `path`{image} ``,** despite fitting the decorator notation exactly. A decorator must mean the same thing in both positions, and ` ```image ` is meaningless: a word that works only inline is a special case wearing the notation. Worse, a decorator labels content that is *present*, where a path references content that is *absent* — the very shape decision 15 forbids. And it has nowhere to put alt text, so a plain-text reader gets `bridge-dusk-2019.jpg` where `![The Alexandra Bridge at dusk](…)` gives them what the picture shows.

    **Rejected — `[alt](src){image}`,** the strongest alternative, which fails on cost rather than on principle. The principle favours it: `![]()` is the only *inline* construct whose type marker precedes its content, and prefix markers were rejected everywhere else inline. But decision 12 forbids diverging from an unambiguous, harmless CommonMark construct for legibility alone; it opens a third decorator position, growing the rule rather than shrinking it; and it would make nearly every real document invalid CommonMark, since almost every README carries badges or screenshots. That last cost also destroys the self-hosting measurement, which exists precisely to count the documents that genuinely need `.markleft`.

    **The line, tested — file includes stay forbidden.** Could `![…](chapter-2.md)` include a text file in place, with `[…]` linking to it? **No**, and the rule decides it without needing an amendment, which is the test of an invariant rather than a carve-out. An include inserts text the source does not contain, so a plain-text reader has *not* read the document: it fails exactly where an image passes, and it fails the same way `<iframe>` does — arbitrary imported text rather than a bounded illustration. The prohibition is over-determined besides, since include mechanisms are what document-preparation systems have (AsciiDoc `include::`, MyST, Quarto, mdBook), and the charter-level non-goal already excludes that product category. **The legitimate want behind it** — keeping a code sample in sync with the file it came from — is real, and gets the same answer tables of contents got: a preprocessor or the editor writes the snippet *into* the source, where it becomes plain text like everything else. An include directive hides drift; an inlined snippet shows it in the diff.

    *Left open deliberately:* whether decorators gain a third position, so that `## Heading {#custom-id}` becomes expressible. `#id` exists for anchors and cannot currently reach a heading, which is a real gap — but it should be settled on heading anchors' own merits rather than as a side effect of image syntax.

17. **Anchors are positional; references are ordinary links.** *(Decided 2026-08-07.)* `{#id}` is valid **anywhere** and marks a **point** in the document rather than decorating a construct. There is no attachment rule, so paragraphs need no exclusion, mid-sentence anchoring is free, and the adjacency subtlety that an attachment rule would have required never arises. **The rule that places it: an anchor marks where the reader arrives.** A link scrolls its target to the top of the viewport, so an anchor at the end of a paragraph delivers the reader past what they were sent to read — lead for anything longer than a line (`{#five-minute}The complete core fits…`), trail on a heading (`## The five-minute property {#five-minute}`), and neither is a special case: both follow from asking where the reader should be looking. Two authors anchoring the same paragraph at different points is a stylistic difference, not an ambiguity — one parse either way, so invariant 3 is untouched. **Whitespace around an anchor is optional and insignificant**, on both sides, because whitespace never carries meaning here (decisions 10 and 13) and no reader can perceive a point sitting before a space rather than after it. Requiring the space would make `{#id}Text` fall back to literal text under the uniform fallback rule, and forbidding it would spring the same trap on the other natural spelling; neither buys a second meaning. A present space is ordinary inline text, so the two spellings differ in the AST and render identically — the same situation as a labelled versus unlabelled span, and decision 15 requires the AST to record what was written. Normalization is a Phase 3 formatter question, like token order.

    **References need no new syntax at all, and that is the whole point.** `[text](#five-minute)`, `[text](charter.md#five-minute)`, and `[text](https://example.org/c#five-minute)` are one construct — a link whose target is a URL carrying a fragment. Internal and external references are therefore consistent for free, because URL semantics already unified them. **Markleft's contribution to cross-referencing is the removal of a heuristic, not the addition of a feature:** platforms derive anchors by slugifying heading text and every platform slugifies differently, so the same link resolves on one and dangles on another — the founding exhibit in navigation form. Explicit anchors delete the guess.

    **Rejected — a bare reference sigil,** in every spelling considered (`#id`, `@id`, `@#id`). Two independent failures. It would have to **generate its own text**: rendering `@#five-minute` means emitting either the useless literal `five-minute` or the target's title or section number, which is text pulled from elsewhere and inserted where nobody wrote it — decision 15, arriving through a smaller door than a table of contents but through the same door. And bare `#id` is **prose-unsafe**: `#general`, `#nowplaying`, and `#hashtag` are ordinary technical prose, and it would make `#word` text at the start of a line (decision 4) but syntax in the middle of one. `@` collides with handles for the same reason; `@#` escapes the collision only by being obscure enough that nobody would type it, which is also why nobody would reach for it. *(No digit-guard rule is needed as a result: with references written as links, `#1` in prose is never syntax and never needs escaping.)*

    **The validator stays a single-document tool.** Duplicate ids and dangling same-document fragments are lint. A target carrying a path or a host is a **URL**, and Markleft does not validate URLs — an `https://` link can 404 and is not checked either, so a cross-document fragment gets the same treatment. This keeps the tooling from needing to know about a document set, which would have been its first multi-file commitment.

    **Cost, stated:** `{#` becomes structural in prose, so Svelte and Handlebars snippets (`{#if}`, `{#each}`) need `\{` outside a fence. Decision 3's uniform escape covers it and no new rule is required, but it is a real papercut for one audience.

## 5. Name: Markleft

Cardinal-point extension of markup/markdown. Four readings, all load-bearing:

1. **The compass:** orthogonal move — same size as Markdown, different rigor.

2. **What's left:** the language is what *remains* after 15+ years of curation — a subtractive philosophy (most of the binding decisions remove something).

3. **We left:** departure from the informal era (chosen over Marklock's enforcement energy and Markright's self-verdict pretension — the fourth direction stays unclaimed, left to whoever comes next to do it "right").

4. **Left margin:** the natural state of plain text — prose-safety subliminal.

**Copyleft lineage (structural, not just aesthetic):** copyleft used copyright's machinery to guarantee openness; Markleft uses formal-spec machinery to guarantee prose freedom. The formalism exists so plain text stays plain.

**The self-installing meme:** a stranger asks "What's `left`?" — and the question is the elevator pitch. Enshrined.

**The two axes of the naming family:** Markdown-adjacent names sort onto two morphological axes. *Suffix-keepers* vary the prefix and keep "-down" (Showdown, Kramdown — "Markdown" reversed — Quarkdown, whose name is a physics pun: quarks come in flavors, and Markdown dialects are literally called flavors). Keeping the suffix signals *membership*: "another flavor of Markdown." *Prefix-keepers* hold "mark-" and vary the direction (markup → markdown → Markleft). Keeping the prefix signals *succession*: the next move in the sequence, not another dialect of the ambiguous thing. Markleft is deliberately on the succession axis. (Charter Name section material.)

**Name graveyard (considered, rejected — kept for the history):**

- **Doubledown** (floated 2026-08-06) — quirky, memorable, and genuinely funny; rejected on the two-axis argument above. It is a *suffix-keeper*, so it signals membership ("another flavor of Markdown") when the entire positioning is succession — and it drops "mark-" as well, taking the membership signal without even the family recognition, which is the worst square of that grid. The semantics run backwards too: doubling down means committing harder to a position already held, where this language is subtractive. And a gambling metaphor on the masthead of a formal standard is a fight you would have to keep winning forever. *Survives as rhetoric rather than as a name:* CommonMark doubled down — it standardized the accident instead of fixing it. Charter prior-art material, where it works far better as the thing we contrast against than as our own banner.

- **Marklock** — enforcement energy. The formalism exists so plain text stays plain; nothing is locked.

- **Markright** — self-verdict pretension; a name should not award itself the correctness it has yet to demonstrate. So the fourth direction is left open on purpose: the next iteration, not this one, gets to claim it did it "right".

"One-meaning property" survives as the internal name of the core guarantee.

## 6. Extensions (DECIDED — both forms)

Exactly two extensions ship: a one-meaning language shouldn't have four names for its files.

- **Long form (decided):** `.markleft` — canonical in documentation; the stable anchor and the collision-proof long-term claim, unambiguous in any namespace regardless of what the short form does.

- **Short form (decided — `.lf`):** the `.md` two-letter family echo and the line-feed pun carried the call. Rejected alternative `.lft` had a marginally cleaner spoken form ("el-eff-tee file" gets a stop-consonant buffer before "file", where "el-eff file" can smear eff-into-file) and a clean namespace (only obsolete claimants: ChiWriter DOS fonts from 1986, an obscure CAD loft format) — it is the designated fallback if `.lf` ever proves untenable, not a live option.

- **Graveyard (collisions & retirees):** `.lft` (runner-up, held in reserve), `.left` (bare English word, too collision-prone), `.mklf` (safety candidate, retired — its insurance role is covered by `.markleft`), `.ml` (OCaml), `.mll` (ocamllex — ironic: lexer files), `.mlf` (unfortunate acronym; also HTK), `.mlt` (MLT video framework), `.mlk` (spoken collisions), `.mkt` (market), `.mf` (Metafont — never disrespect Knuth).

## 7. Copyright & licensing

*Corrected 2026-08-06 — see §11. This section previously described an institutional home and steward. That was wrong and is removed, not annotated.*

- **Copyright:** Crown copyright, **by operation of law and not by choice**. An author is a Canadian federal public servant, so Copyright Act s.12 applies whatever anyone would prefer. It is not sponsorship, endorsement, review, or stewardship. No institution governs this project or vouches for it, and no document may imply that one does.

- **Metrology framing:** borrowed as an intellectual model, never as provenance — one definition, many independent realizations, kept honest by comparison. The conformance suite is a key comparison for parsers; the canonical formatter is a reference realization. The framing earns its place on the argument alone.

- **Spec + conformance suite:** CC0 *intent*; instrument pending legal review (Crown copyright s.12 vs CC0-as-waiver interacts non-trivially; fallback formulation "CC0 where applicable; otherwise licensed without restriction"; OGL-Canada is the historical departmental route). Exact notice wording is for counsel; everything currently in the repos is a placeholder.

- **Code:** MIT.

- **Conformance mark:** "Markleft-conformant" reserved as a possible lightly-governed mark (Phase 4; reserve the possibility in the charter now) — held by the project's maintainers, not by any institution.

- Inverse-Markdown posture: hold the language tightly, give the name and the spec away freely.

## 8. Name availability accounting (done)

- **npm `markleft`:** SQUATTED — abandoned 2013 "simplest markup language ever" (Fovea, v0.3.0, MIT). Same name, same category, dead. → Use scope **`@markleft/*`** (verify scope availability); bare name recoverable via slow npm dispute, non-blocking.

- **GitHub:** bare `markleft` account TAKEN (moribund). → **`markleft-lang` org PARKED ✅** (rust-lang precedent). Bare name flagged for possible future trademark-process recovery; org renames redirect, so migration stays open.

- **LaTeX:** `\markleft` is a KOMA-Script page-heading macro (counterpart of `\markright`). Low severity; charming irony; footnote material.

- **Domain:** markleft.com parked/buyable at GoDaddy. markleft.org apparently undeveloped. `.ca` fits the Canadian origin.

- **Trivia tier:** Botswana print shop, `markleft.ttf` font file, 11-follower Instagram. Irrelevant.

- **Trademark registries:** no registered mark surfaced in web search; formal formal CIPO/USPTO/EUIPO search via counsel = OPEN ITEM. Note for examiner: the 2013 npm package is weak, non-commercial, non-continuous prior use.

## 9. Roadmap

- **Phase 0 — Charter & corpus.** Write the charter (invariants + binding decisions with rationales and rejected alternatives). Seed the prose corpus in `tests/corpus/` — contributor-authored documents written to attack invariant 1. *Amended 2026-08-06 (§11): originally specified collected corpora of real READMEs and LLM output, with "% of real documents rendering identically" as the disciplining metric. That measurement is no longer a deliverable.*

- **Phase 1 — Grammar & spec-as-tests.** PEG for inlines; explicit small-step block algorithm (honest about the non-CFG bits like fence-length matching); exhaustive conformance suite in CommonMark `spec.txt` style. *Amended 2026-08-06 (§11): originally "the suite IS the standard". The document is normative; the suite is the key comparison. See `notes/normative-hierarchy.md`.*

- **Phase 2 — Reference parser & canonical AST.** Rust (speed, WASM playground, FFI). Specified JSON AST with source positions; the AST spec is a deliverable.

- **Phase 3 — Killer tools.** Validator with real diagnostics ("unclosed emphasis opened at 12:4"); canonical formatter (gofmt-style, one serialization per AST); CommonMark→Markleft migrator with change report (the adoption bridge).

- **Phase 4 — Ecosystem.** One-page cheat sheet; browser playground; editor highlighting; conformance badge; a system-prompt-sized language description so LLMs can emit valid Markleft (the network-effect wedge AsciiDoc never had).

## 10. Open items checklist

- [x] GitHub org — **markleft-lang parked**

- [ ] npm scope `@markleft` — verify and claim

- [ ] crates.io `markleft` — verify and claim

- [ ] Domain — pick and acquire (markleft.org / markleft.ca candidates)

- [ ] Trademark search via counsel (CIPO, USPTO, EUIPO)

- [ ] CC0 instrument legal review (Crown copyright interaction)

- [ ] Draft the Phase 0 charter (public-standard voice from day one — a standard has to read like one), including the explicit non-endorsement statement

- [ ] Seed `tests/corpus/` with contributor-authored prose

- [ ] Create the `tests` and `.github` repos under the org

## 11. Amendments

Decisions recorded above are historical; amendments are appended here rather than edited in place, so the record shows what changed and when.

**2026-08-06 — repository layout and corpus.** Full memo: `notes/repo-layout.md`.

- Three repos under `markleft-lang`, not one: `markleft` (spec + implementations

  + tooling), `tests` (conformance suite + prose corpus, separately vendorable), `.github` (org profile). The org is the top-level home; nothing sits under a personal account.

- The standard is held alone — no tests, no data, no tooling — leaving the promotion path to a standalone spec repo open. *(Superseded 2026-08-07: the promotion happened, and the `spec/` subdirectory it referred to no longer exists.)*

- The collected corpus is dropped. `tests/corpus/` holds only contributor-authored prose; nothing is scraped, imported, quoted, or adapted, so the suite carries a single licence and no third-party redistribution question arises.

- No governance repository. Governance *posture* is a charter section; governance *process* goes to `.github` when a first outside contributor appears; a dedicated repo splits out on the same trigger as `rfcs` — more than one decision-maker.

- Each repo carries its own `CLAUDE.md` and sessions launch from inside a repo. The org folder is not a repository, so anything placed there is uncommittable and machine-local.

- **Cost of that change, recorded deliberately:** §9's "% of real documents unchanged" metric was to be the empirical check on decisions 4, 5, and 6, the three flagged as carrying the most muscle-memory risk. An authored corpus cannot supply it — we would be writing the documents our own rules already handle. That evidence now arrives in Phase 3 via the migrator's change report, when acting on it is more expensive. If this proves uncomfortable, the recovery is a manifest-only corpus: URLs and content hashes committed, documents fetched locally and never redistributed.

**2026-08-06 — normative hierarchy.** Full memo: `notes/normative-hierarchy.md`. Charter material.

- The **specification document, grammar included, is normative.** The conformance suite is the **key comparison**: mandatory, exhaustive, and binding on any conformance claim, but subordinate to the document. Realizations — the reference parser included — are privileged over nothing.

- Supersedes "the suite IS the standard" (§9, Phase 1), inherited from CommonMark along with the spec-as-tests methodology. Only the normative claim is amended; the methodology stands. CommonMark needed normative examples because it had no grammar; we are paying for formalization and should collect the benefit.

- §7 already had the right term — the suite is "a key comparison for parsers" — without following it through. A key comparison is not a definition.

- **Precedence rule.** When suite and document disagree the document governs and the test is presumed the bug, *unless* the test is evidence the document needs revising — a spec revision through this record, never a quiet edit. The rule settles where a change must land, not who was right. What it forbids is a behavior existing in the suite and nowhere in the document.

- Consequence: conformance examples assert against the specified JSON AST, not against HTML. CommonMark's `spec.txt` asserts HTML and thereby binds the standard to a serialization unrelated to the language — the main reason test suites age badly, and the drift this amendment exists to avoid.

**2026-08-06 — no institutional stewardship.** Corrected in place throughout, rather than annotated, because the original framing was a misstatement of fact and leaving it visible would keep propagating it.

- Markleft has **no institutional steward, sponsor, endorser, or governing body**. It is an independent project maintained by its authors.

- **Crown copyright applies by operation of law** — an author is a Canadian federal public servant, so Copyright Act s.12 attaches regardless of preference. A copyright notice is not sponsorship and must never be written so a reader could infer that it is.

- The **metrology framing stays**, as a borrowed intellectual model that earns its place on the argument: one definition, many independent realizations, kept honest by comparison. It is not provenance and confers no authority.

- Removed accordingly: "stewarded by" in the project description, the Stewardship section of the org profile, "NMI stewardship" as a claimed delta over djot, "disinterested institutional steward" in the positioning summary, institution-named copyright markers, "NRC provenance" as the justification for the charter's voice, and the institutional premise of the argument for using a GitHub org.

- The org-profile README carries an explicit non-endorsement note, since that page is where a reader is likeliest to mistake a copyright line for a backer.

**2026-08-07 — repository layout follows the normative hierarchy.** Revises the 2026-08-06 layout entry above. Full memo: `notes/repo-layout.md`.

- One repository per layer: `markleft` is **the standard itself** (CC0), `tests` is the key comparison (CC0), `markleft-rs` will be the Rust realization (MIT, Phase 2), `.github` is the org profile. The namesake repository holds the definition, not the code.

- Supersedes the working-monorepo shape, which predated the normative hierarchy. Once the document became normative, filing it as one directory inside a repository named after its own implementation stopped making sense — and it left `markleft` mixed-licence, the incoherence that prompted the question.

- Nothing had to move: every implementation directory in the old layout was still hypothetical.

- **No `spec/` subdirectory** (settled 2026-08-07, same day, after one round of getting it wrong). The standard sits at the repository root, so no URL or citation contains `markleft/spec/`. Keeping a `spec/` directory would have reintroduced the monorepo framing — the standard as one component among several — inside the very decision that removed it.

- The licensing boundary a `spec/` directory would have drawn is drawn by exclusion instead: **everything in `markleft` is the standard except `CLAUDE.md` and `.claude/`**, stated in `README.md`. The exclusion list is short and stable, which makes it cheaper than a directory that appears in every path forever.

- **`markleft-<lang>` is reserved for independent implementations** (`markleft-py`, `markleft-cpp`). A binding is not a realization: wrappers over the Rust core live in `markleft-rs/bindings/`. Running the suite against a wrapper compares a realization with itself — it measures nothing while looking like a passing key comparison, and an ecosystem of five bindings and one parser has one implementation.

- The pairing burden between spec and suite is unchanged by this: it exists between `markleft` and `tests` wherever the spec lives. `markleft-rs` needs no pairing, because an implementation follows the standard rather than co-evolving with it.

**2026-08-07 — Markdown and Git conventions.** Recorded in `CLAUDE.md`; applied across all three repos.

- Markdown: no hard wrap (one line per paragraph), list-item spacing decided per list rather than per item, blank lines around every heading, list, fence, and table. All nine existing files were reflowed in one pass; a wrapped heading in `landscape.md` that had been half-heading and half-prose was fixed in the process.

- The no-hard-wrap rule matters more here than in most projects: this record is edited by amendment, so a word-level diff *is* the audit trail. Reflowing a wrapped paragraph rewrites every line and buries the actual change.

- Git: the cbea.ms seven rules in full (blank line after subject, subject ≤50 characters, capitalized, no trailing period, imperative mood, body wrapped at 72, body explains what and why). Atomic commits, sequenced so the history tells the story, staged and shown as a diff before committing.

- Also: keep a "Claude" co-authorship credit but strip the `<noreply@anthropic.com>` address, so platforms cannot render it as a link to a dead mailbox — accepting that the trailer then reads as text rather than parsing as machine-readable co-authorship. No Claude-Session or `claude.ai` URLs in commit messages. No merge bubble around a single commit; `--amend` only against an unpushed tip; no bare `git stash`, whose stack is shared across worktrees.

- Note the deliberate asymmetry: commit bodies wrap at 72, Markdown prose does not wrap at all. Different media, different rules — do not let one convention "fix" the other.

**2026-08-07 — verbatim guarantee, stated publicly ahead of the record.** Added as the fifth guarantee in both READMEs. **Two of its clauses are new normative commitments and need to become numbered decisions in §4 before the charter is drafted.**

- **Escaping extended explicitly to line endings.** Decision 3 already says a backslash before *any* character yields that character with no exception list, so line endings are arguably covered. Making it explicit is still worth it, because line endings are exactly where engines diverge today, and a guarantee that has to be *inferred* is not the kind this project makes.

- **A backslash before a line ending is the line break.** Explicit and visible in the source. **Settled same day as decision 13: the trailing-space hard break is removed.** The deciding argument is prose-safety rather than aesthetics — two spaces after a period is a live typing habit forty years after typewriters, so a sentence that happens to end a line silently becomes structure. Ordinary prose turning into markup by keystroke habit is invariant 1 failing, not a style quibble. Everything else compounds it: editors and hooks strip end-of-line whitespace to keep diffs clean, so the syntax is not merely invisible but actively unstable. (The one language that made whitespace load-bearing on purpose, Whitespace, is a joke — deliberately.)

- **Unicode is entirely new to this record.** The full Unicode range is text, carrying the same verbatim guarantees as content and in every construct that takes text. This was not in §3 or §4 in any form. It needs a numbered decision, and the charter needs to answer what the one-sentence guarantee does not: the **document encoding** (UTF-8 is the obvious answer, but it is a separate commitment from the character-repertoire claim and has not been made), **normalization** form and whether any is imposed, **identifier and anchor handling**, and whether "the same guarantees" attach to a code point or a grapheme cluster.

- Keep the public wording at the level of the repertoire — "the full Unicode range is text" — and leave UTF-8 to the specification. The splash page should not pin an encoding the record has not decided. State it positively, too: a guarantee phrased as the absence of a limitation ("has never meant ASCII") reads like a deletion notice rather than a promise.

- **Note on the count.** The four invariants in §3 remain four; they are constitutional. The READMEs list five *guarantees*, which are user-facing promises derived from the invariants and decisions. Do not "fix" the mismatch by adding a fifth invariant.

**2026-08-07 — "what Markleft is not", stated publicly.** Recorded in both READMEs and in `CLAUDE.md`'s non-goals. Charter material: this belongs in the charter's non-goals section.

- The existing non-goal ruled out a *different product* — toolchains and document-preparation systems. This adds the narrower and likelier misreading: that Markleft is a **richer Markdown**. It is not, but neither is it an austerity project, and the docs must not claim otherwise — math, attributes, pipe tables, and the raw-HTML block are all additions.

- **The rule:** every addition must remove an ambiguity or fill a genuine gap, *and* cost nothing in plain-text clarity. A construct that makes ordinary unmarked prose read worse is declined regardless of merit — prose-safety is invariant 1; richness is not an invariant at all.

- **The budget:** the five-minute property. Most decisions remove, a few add, the total fits on one reference card. "Markleft could also do X" is not an argument; the argument must be that X is worth what it costs every reader who never uses it.

- djot remains the right place to shop — most of our syntax is djot-vetted — and this is the test applied to djot features we have not taken (smart punctuation, definition lists, generic containers).

**2026-08-07 — self-hosting as a stated goal.** Recorded in `CLAUDE.md`.

- Every document the project ships should be written in Markleft — self-hosting in the compiler sense, applied to documents. Stronger than dogfooding, because it has a finish line and, from Phase 3, a test.

- Staged: Markleft-valid content under `.md` now; validator-enforced in CI at Phase 3; `.markleft` sources with generated `.md` companions at Phase 4, but only for documents that need syntax CommonMark lacks.

- **GitHub renders `.md` and will not render `.markleft`**, so `.md` files stay permanently. The only question is whether they are authored or generated, and generated files are a maintenance tax not worth paying for documents that could have stayed valid in both.

- **New cheap metric:** the fraction of our own documents that *require* `.markleft` measures how far Markleft diverges from Markdown in practice, and tests decision 12 directly. It partially recovers what was lost with the collected corpus (§11, 2026-08-06) — and unlike scraped material, we can publish every byte of our own repos.

**2026-08-07 — ATX headings closed to one form.** Riders added to decision 4 (§4); three of them are new deltas from CommonMark (`deltas.md` rows 6–8, which renumber the rows below them).

- **One form, one fallback.** A heading is one to six `#` in the first column, one space, then non-empty text; a line that does not match is text. The uniform fallback disposes of `#######`, `#hashtag`, `#5`, and the lone `#` with a single rule, no "unless" clause, and no new error class — which is why the empty heading is a paragraph rather than a rejected input.

- **Removed:** the optional closing run (`## Title ##` now has the text `Title ##`, and CommonMark's rule for when a trailing run closes is the clearest available example of the clause invariant 2 forbids); the up-to-three-space indentation allowance, obsolete the moment decision 5 deleted indented code; and the empty heading.

- **The six-level cap stays, with its reason now on the record:** HTML has `h1`–`h6`, the other targets agree, and CommonMark caps there too — so the cap costs no delta under decision 12, where raising it would buy nothing any renderer could express.

- **`#` meaning "number" needs no special case.** The required space already makes `#5` text at the start of a line; only `# 5 is my favourite` needs decision 3's escape. Prose-safety here is structural rather than a carve-out, which is the shape every such rule should have.

**2026-08-07 — UTF-8 confirmed; the durability risk relocated.** Memo: `notes/unicode-and-utf-8.md`. Decision 14 unchanged.

- UTF-8 is the terminal answer rather than an interim one: the WHATWG Encoding Standard requires it exclusively for new formats, RFC 3629 froze its definition, deployment is ~99% of the web, and no successor exists or is proposed. Its age is the argument in its favour — a specification wants a target that has stopped moving.

- **The version, not the encoding, is what can age.** UTF-8 is frozen; Unicode releases annually. So the risk attaches only to rules that consult a Unicode table, and "content is never normalized" already immunizes the bulk of the language.

- Two open riders do consult tables and are therefore one-meaning questions across *time*: case-folding in anchor and label matching, and East Asian Width if "column" is defined by display width. The memo recommends defining both version-independently — code-point identity, code-point count — rather than pinning a Unicode version, so the spec does not inherit someone else's release schedule.

**2026-08-07 — no generated content.** New decision 15 (§4). Recorded in `CLAUDE.md` (core decisions and the non-goals paragraph), `charter.md` §2, and `.claude/landscape.md`.

- **The cardinal rule:** a reader can always read the entire document in plain text, with no compiler, viewer, or extension. Nothing renders that is not in the source, in the order it appears there.

- Declined accordingly: TOC directives, section auto-numbering, generated indexes, transclusion, variable substitution. The `{.toc}` instruction AsciiDoc, MyST, and Quarto provide is useful, and that is why it needs a rule rather than taste — a directive whose payload exists only after rendering makes plain-text reading conditional on owning the tool.

- **This is "not a toolchain" becoming a syntax rule** rather than a scoping statement. The non-goal said what the project would not build; decision 15 says what the language may not express, which is the enforceable half.

- **Back door closed:** decision 9's attributes carry metadata and may never name a content-emitting construct. Otherwise the closed extension namespace would readmit exactly what this decision excludes.

- Tables of contents keep two good homes outside the specification — the editor, which generates one *into* the source where it becomes ordinary text, and the publishing pipeline downstream.

- **Turned up an inherited violation:** CommonMark renumbers ordered lists from the first marker, so `1.` repeated renders 1, 2, 3. That is generated content in the core we inherit. Flagged as an open rider on decision 6, not settled here.

**2026-08-07 — decision 15, boundary stated precisely.** Same day, refining the entry above.

- The requirement is **semantic equivalence up to format**, not visual resemblance. `**bold**` is not bold and `\(\frac{a}{b}\)` is not a typeset fraction; both pass, because what they mean is written down and a reader parses it from the plain text without running anything.

- **Rendering may change how written content appears; it may never create content.** Reference links pass — the visible text is written where it appears, and only the target lives elsewhere, a target being metadata rather than content. A table of contents fails: its entries appear where nobody wrote them.

- The test for any future construct is one question: *is this changing the presentation of written content, or manufacturing content?* Presentation instructions stay admissible in principle — decision 9's attributes are one — and how far they may go is deliberately left open. Manufacturing is closed permanently.

**2026-08-07 — inline math becomes a decorated verbatim span.** Decision 1 revised (§4); `\(...\)` withdrawn in favour of `` `x=y`{math} ``. Delta 2 updated.

- **`\(...\)` was unspecifiable.** Decision 3 yields the character after any backslash with no exception list, so `\(` is a literal `(`. The two decisions could not both stand, and decision 3 is the one carrying the promise.

- **The content was the deeper problem.** Inline TeX is backslash-dense: `\(\frac{a}{b}\)` loses `\f` to an escape, and nothing distinguishes a closing `\)` from an escaped paren. Inline math must be **verbatim**, not escaped text — which is why the fenced form was never at risk, fences being verbatim by construction.

- **So inline math is not a delimiter problem, it is a tagged-verbatim problem** — and both halves were already on the record: backtick verbatim (decision 11) and the decorator grammar (decision 9). The inline form needed no new syntax, only the statement that the registry is reachable inline.

- Rejected: carving `\(` and `\)` out of decision 3. It costs the "no exception list" invariant *and* does not fix the content problem, so it buys nothing on its own.

**2026-08-07 — raw passthrough removed entirely.** Decision 7 reversed (§4). Delta 11 rewritten; `.claude/landscape.md`, `README.md`, and `CLAUDE.md` updated.

- **Decision 15 is the reason.** `<iframe>`, `<img src>`, `<script>`, and `<object>` render content that is not in the source. That is transclusion, which the cardinal rule forbids by name, so any document with a raw block falsified "you can read the whole document in plain text" by exactly the mechanism decision 15 exists to exclude.

- **The hole could not be patched by rule.** `<br>` adds nothing, `<iframe>` adds everything, and telling them apart needs HTML semantics — an unbounded blocklist and an "unless" clause at once. Invariant 2 refuses it, so the choice was binary.

- **Bought:** a document **inert by construction** (no execution, no embedding, ever — a claim no Markdown flavour can make, and worth more in the LLM-emitter argument because "safe" needs no qualifier); no target-format lock-in, `=html` having put HTML inside a language that claims independence from any serialization; and a simpler decorator grammar, since the `=name` token shape leaves with it.

- **Paid, and stated in the decision rather than hidden:** an escape hatch is what lets a small language refuse feature requests. Without one, every gap presses on the core — the force that killed the five-minute property everywhere else. `<details>`/`<summary>` is the concrete casualty; djot answers it with generic containers, which we decline.

- Rejected: keeping the raw block as a loudly-marked exception. Honest, but still an "unless" clause on the cardinal rule.

- **Deferred deliberately:** what the migrator does with HTML in existing documents. Principle first; the migration story is a later problem and does not change the design.

**2026-08-07 — attributes reduced to decorators.** Decision 9 rewritten and renamed (§4). Delta 13 updated; terminology changed in `CLAUDE.md`, `README.md`, `.claude/landscape.md`, and decision 15.

- **The name is load-bearing.** *Decorator* evokes style and rendering, which is exactly what these do and all they may ever do. *Attribute* suggested arbitrary key-value metadata, which is the thing being removed.

- **Three token shapes, no zoo:** a bare word naming the format (at most one), `.class`, and `#id`. `key=val` is gone with the raw HTML that motivated it — arbitrary HTML attributes only ever existed to feed arbitrary HTML, so decisions 7 and 9 removed each other's reason to exist. Class and id keep their sigils because they name things that are not formats.

- **One vocabulary, two positions**, which collapses the previous two extension points into one: the same token list is written after a fence's opening backticks or in braces after an inline span's closing run. ` ```math ` and `` `x=y`{math} ``; ` ```.note ` and `` `x`{.note} ``. The registry is now the single place the language can grow.

- **`text` names the default explicitly**, idempotent with the bare fence and bare span. Two spellings of one meaning is permitted — invariant 3 forbids one spelling with two parses, not the reverse — and saying "this is plain text" out loud is the whole point of a decorator.

- **Unknown words fall back to plain verbatim and warn on lint; they never fail.** The split matters: the *language* defines `math` and `text`; the *validator* carries a list of recognized words that grows without a specification revision. ` ```rust ` therefore keeps working and stays byte-compatible under decision 12, a typo still gets a diagnostic, and the normative document stays small. The label is retained in the AST because decision 15 lets a renderer consume a mark but never lose one.

- Flagged for the grammar: decision 2's `{*...*}` is not a decorator. They cannot collide — a decorator follows a backtick run and never opens with `*` — but the grammar must state it rather than leave it to inspection.

**2026-08-07 — decision 15, the back half.** Added to §4 decision 15 and summarized in `CLAUDE.md`. Completes the rule; nothing above is reversed.

- **Rendering is lossy on marks and lossless on content.** The front half forbids adding. The converse is that rendering *removes* the mark: `` `E=mc^2`{math} `` becomes a typeset equation and the word `math` leaves the page. Nothing is lost, because "this is an equation" was re-expressed as typography rather than deleted — the same trade `**bold**` makes when its asterisks surrender to boldness.

- **A decorator is the plain-text stand-in for a visual distinction** the plain-text reader cannot receive. That is why these are two halves of one rule: the reader who gets no typography gets the word instead.

- **The constraint on this direction:** only marks may be consumed, and every unit of content survives. Dropping the content a mark identified breaks the rule as surely as inventing content does.

- **Design test that falls out:** every format word the language defines must name something a conforming renderer visibly distinguishes. A format rendering indistinguishably from undecorated content destroys information and is refused. Classes and ids are exempt by kind — CSS linkage and anchors are metadata, the same category as a reference link's target, and metadata has no content to lose.

**2026-08-07 — no word is ever reserved.** Decision 9 reduced again, same day. `math` and `text` leave the specification; `.class` and `#id` are the only tokens it assigns meaning to.

- **The registry was the wrong shape.** `math` and `text` are format labels exactly like `rust` and `python`; privileging two of them put an arbitrary word list in the normative document and gave the language a way to grow by accident.

- **What replaces it is stronger than what it replaces.** The specification defines the grammar and nothing about any word's meaning, so **no decorator word can affect the parse.** The grammar is fixed independently of any word list — no future word can quietly become a directive, because words have no parse-level power by construction, and there is no list to maintain, version, or argue about.

- **Meanings move outside the standard**, to a non-normative conventional-labels appendix and the validator's recognized-word list, both revisable without a specification revision. Unknown words still parse as plain verbatim and warn on lint.

- **Multiplicity, decided:** at most one bare word (kebab-case permitted), at most one `#id`, and **any number of `.class`** — the single multiplicity exception, because an element genuinely carries several classes and stylesheet languages are built on that. Order is free; the canonical formatter normalizes it.

- **Consequence:** ` ```text ` and a bare fence are no longer the same node. They render identically, but one carries a label and the AST records which the author wrote, as decision 15 requires. "Idempotent" now means renders-identically.

- **New open rider:** what character set a word, class, and id may use. Decision 14 forbids privileging ASCII in constructs that take text; a label is an identifier rather than text, and the charter must say which rule governs.

**2026-08-07 — what "math is core" actually claims.** Decision 1 restated (§4), following the removal of reserved words. `README.md` and `CLAUDE.md` updated. No syntax changes.

- **Three guarantees, and only three:** `$` is never math syntax anywhere; a first-class construct exists at both block and inline sizes with **verbatim** content, safe from reinterpretation; and its label reaches the AST intact.

- **Those three are exactly what the founding exhibit needed.** What failed in §1 was that `$` paired across a paragraph and that code spans did not protect the content — not that "math" lacked a definition. The word was never load-bearing, which is why it costs nothing to move it out of the specification.

- **The goal was unequivocalness, not a math engine.** *(Author's framing, recorded verbatim in intent.)* The language guarantees what the source unambiguously is; implementation of the rendering stays free. `` `…`{math} `` is the right answer because it draws that line in the syntax itself.

- **The README claim was reworded** to match. "Math is core rather than a renderer's afterthought" was loose enough to imply the specification defines the rendering; it now names the three guarantees and says outright that a renderer still chooses the typesetting but can no longer change what the source says.

**2026-08-07 — decorator multiplicity, with its reasons.** Added to decision 9 (§4). Confirms the rule rather than changing it; adds one new call on id uniqueness.

- **At most one word, at most one `#id`, any number of `.class`** — confirmed as stated.

- **Why one word:** two format words would claim two content types for one box. There is no meaning to combine them into and no resolution order to appeal to, so a second word buys a race and an undecidable decoration rather than a capability. *(Author's reasoning, recorded because it is stronger than "it is simpler".)*

- **Why one id, and why any number of classes:** this is HTML's own multiplicity. An element carries a single `id` attribute where `class` takes a space-separated set, and `#foo` selects one element where `.foo` selects a set. The decorator therefore maps onto HTML and CSS without translation, and restricting classes would break the thing classes are for.

- **New call — duplicate ids are a lint check, not a parse error.** HTML additionally requires ids to be unique document-wide, which the grammar cannot express without abandoning the local, single-pass discipline invariant 4 buys. A duplicate id is well-formed syntax and a broken anchor, so it joins unknown words in the validator rather than in the parser.

- **Legibility of `` `x`{math .display #eq-euler} ``:** the ceiling of the grammar, not its normal use, and admittedly at the edge of comfortable running prose — a decorator can outgrow the content it decorates. Permitted anyway, because forbidding the full form inline only would be an "unless" clause. House style carries the cost: classes and ids belong on fences; an inline decorator normally carries the word alone. Postfix order is what keeps even the ceiling inside invariant 1 — the content reads first and the decoration is a suffix.

**2026-08-07 — the `text` label is withdrawn.** Reverses the rider recorded earlier the same day. Decision 9 (§4) and `CLAUDE.md` updated.

- **The bare forms are defined, not defaulted.** A fence with no decorator and a span with no decorator mean **verbatim plain text, period** — the definition of the unlabelled construct, not a fallback from a missing label.

- **There is no `text` label.** Not forbidden — no word is ever reserved, and forbidding one would reserve it by the back door — but never blessed: the specification does not mention it, the validator calls it redundant, and the canonical formatter strips it. The parser stays total, and the judgment sits with the tools, as it now does for unknown words and duplicate ids.

- **Why it was reversed:** blessing a second spelling costs the "one way to do it" discipline for nothing, and naming a `text` label invites peppering prose with monospace for emphasis — a Markdown antipattern with no reason to be imported. *(Author's reasoning.)*

- **It was also the last reserved word hiding in prose.** Once no word is reserved, a paragraph blessing one particular word was incoherent, so removing it makes decision 9 self-consistent rather than merely shorter.

- The generalized consequence survives: a labelled span and an unlabelled span are different nodes. They may render identically, but the AST records which the author wrote, as decision 15 requires.

**2026-08-07 — backtick verbatim challenged and kept.** Memo: `notes/backtick-verbatim-challenge.md`. Decision 11 unchanged; a pointer added to it.

- **The challenge is legitimate:** the backtick is a dead key on QWERTZ, AZERTY, and Nordic layouts, is visually near `'` and `´`, and is buried on mobile keyboards. Proposed alternative was `|x=y|{math}`.

- **The pipe fails twice over before compatibility is even reached.** It is already the table delimiter (decision 8), and it breaks prose-safety — `|x| < 5`, `P(A|B)`, `a | b` — which is the `$` failure repeated deliberately instead of inherited.

- **No substitute exists, and the reason is structural.** The backtick's inaccessibility and its prose-safety are *the same property*: it is hard to reach because nobody types it in prose, and nobody typing it in prose is what qualifies it as a delimiter. Any character reachable enough to fix the complaint is common enough to re-create the collision. This is the sentence worth keeping from the memo.

- **Answered with tooling instead**, where decision 15 already sends this class of problem: editor snippets, a cheat-sheet line on non-US layouts, and — the sharpest of the three — a validator diagnostic that catches a decorator following a non-verbatim run and asks whether `'` or `´` was typed for a backtick. Lookalike detection at the point of failure, which is what the Typst error-message bar in `.claude/landscape.md` is for.

- **Reopening requires a candidate, not an argument:** evidence that layout friction blocks adoption *plus* a character that survives the prose-safety test. Without a candidate the challenge has nowhere to go.

**2026-08-07 — images admitted; decision 15's line is textual.** New decision 16 (§4). Decisions 7 and 15 corrected in place to match; `CLAUDE.md` updated. No delta in `deltas.md` — the construct is inherited from CommonMark unchanged.

- **The ratio: decision 15 governs the document's *text*.** An image adds none, so a reader of the plain source has read every word of the document and lacks only an illustration. An iframe imports arbitrary text and leaves the reader having *not* read the document; an image is bounded and inert besides. That is the whole distinction.

- **Decision 15 amended in place** from "no file inclusion or transclusion" to "no file inclusion or transclusion of **text**". Without the qualifier it forbade images outright, and the two decisions contradicted.

- **Decision 7 corrected in place.** Its original argument listed `<img src>` alongside `<iframe>`, `<script>`, and `<object>` as content-not-in-the-source. That grouping was wrong — `<img>` belongs with images, not with the transcluders. The outcome is unaffected: the other three are sufficient, and the cannot-be-patched-by-rule argument never depended on the list.

- **Alt text is optional; empty alt is legal.** Mandatory alt would enforce accessibility through the grammar, which invariant 2 would charge for, and the legality of an image never rested on alt text. Missing alt is a lint warning — the sixth judgment routed to the validator rather than the parser.

- **The mnemonic, which is the reason `!` stays:** `[…]` links to the target, `![…]` shows it instead. A mark is cryptic when its meaning is arbitrary; this one is derivable, and nobody had bothered to state the relationship. Same sentence justifies the image under decision 15's back half.

- **Rejected:** `` `path`{image} `` — a decorator must work in both positions and ` ```image ` is meaningless; a decorator labels content that is present, not a reference to content that is absent; and there is nowhere to put alt text. **Also rejected:** `[alt](src){image}`, which principle actually favours — `![]()` is the only inline construct with a prefix type marker — but which decision 12 forbids for legibility alone, which opens a third decorator position, and which would make nearly every README invalid CommonMark and void the self-hosting measurement.

- **Left open:** whether decorators gain a third position for `## Heading {#custom-id}`. `#id` exists for anchors and cannot reach a heading; a real gap, to be settled on its own merits.

**2026-08-07 — file includes confirmed forbidden.** Worked consequence added to decision 16 (§4). No new rule; the existing one decided the case.

- **`![…](chapter-2.md)` as an in-place include fails**, because it inserts text the source does not contain. It fails exactly where an image passes, and in the same way `<iframe>` does — arbitrary imported text rather than a bounded illustration.

- **This is the rule's first live test, and it passed.** The textual line was written for images and then decided an unrelated case with no amendment. That is what separates an invariant from a carve-out, and it is worth recording as evidence that the line is drawn in the right place.

- **Over-determined anyway:** include mechanisms are what document-preparation systems have (AsciiDoc `include::`, MyST, Quarto, mdBook), and the charter-level non-goal already excludes that product category.

- **The legitimate want gets the tables-of-contents answer:** keeping a code sample in sync with its source file is real, and belongs to a preprocessor or the editor, which writes the snippet *into* the source where it becomes plain text like everything else. An include directive hides drift; an inlined snippet shows it in the diff — which is the better outcome on its own merits, not merely the permitted one.

**2026-08-07 — class decorators removed.** Reverses the same day's decision to keep them. Decision 9 (§4) and decision 15's back half updated; delta 13 and `CLAUDE.md` follow.

- **The grammar loses its only irregularity.** Two token shapes remain — a bare word and an `#id` — and **each is at most one**. The multiplicity exception carved for classes is gone, so no rule in the decorator grammar now has an exception clause at all.

- **It failed decision 15's own design test.** GitHub's sanitizer strips class attributes, so the mark is consumed and nothing replaces it — the exact information destruction the back half forbids. The exemption that granted classes a pass was convenient rather than reasoned, and did not survive being questioned.

- **It paid for nothing under the addition bar.** No ambiguity removed, and the only gap filled was "Markdown has no styling" — the richer-Markdown-for-its-own-sake trap.

- **It cost invariant 1 directly:** any non-trivial CSS structure turns readable prose into a scaffold of class names.

- **The deciding argument, and the author's:** it invites mixing styling with content, which is an antipattern here and the very thing many people came to Markdown to escape. Being freed from styling decisions is a feature, not a deprivation — no pixel-perfect subjective loops. Adoption evidence agrees: Markdown's lack of styling softly enforces uniformity, and that uniformity is part of why it pastes anywhere.

- **Consistency:** raw HTML was removed for breaking uniformity of content; classes broke uniformity of presentation. Same refusal, same shape.

- **`#id` survives on a non-presentational argument** — an anchor is navigation, works with zero CSS, and is what makes cross-references deterministic. See the next entry.

**2026-08-07 — decision 16's rationale corrected to name the bend.** Same-day refinement of decision 16 (§4). The rule is unchanged; the justification was overclaiming.

- **What was wrong:** the entry said a plain-text reader "lacks an illustration, not content". An illustration *is* content in the ordinary sense, and no alt text recovers what a picture shows — an alt attribute is sized for a screen reader, not for a thousand words.

- **What replaces it:** the cardinal rule *is* bent for images, deliberately, in one bounded place. Images are essential to clarity in most real documents, and unlike text there is no way to carry one in plain text at all. **A rule bent knowingly, at a named place, with the reason written down, is not a rule with a hole in it** — which is the distinction that matters, and the reason this does not license the next request.

- **The operative test is unchanged and still does the work:** an image adds no *text*. That is what decided file includes against admission, and it decides them the same way now.

- **Base64 embedding** would close even the residual gap, and is worth having as an *optional* shrink-wrapping step — never a requirement. Inlining binary wrecks diffs, makes a lightweight format heavy, and would discourage the image use this decision exists to permit.

**2026-08-07 — anchors positional, references unchanged.** New decision 17 (§4). Decision 9's `#id` bullet points to it; delta 13 narrowed to the word, new delta 18 added.

- **`{#id}` marks a point, not an attachment.** That removes the whole attachment question: paragraphs need no exclusion rule, mid-sentence anchoring is free, and the adjacency subtlety an attachment rule would have forced never arises. Whitespace around an anchor is optional and insignificant on both sides, consistent with decisions 10 and 13.

- **The placement rule: an anchor marks where the reader arrives.** A link scrolls its target to the top of the viewport, so a trailing paragraph anchor delivers the reader past what they came for. Lead for anything longer than a line, trail on a heading — neither a special case.

- **References need no new syntax, and that is the finding.** `[text](#id)`, `[text](file.md#id)`, and `[text](https://…#id)` are one construct: a link to a URL fragment. Internal and external consistency is free because URL semantics already unified them.

- **Markleft's xref contribution is removing a heuristic, not adding a feature.** Slug-guessing differs per platform, so the same link resolves on one and dangles on another — the founding exhibit in navigation form. Explicit anchors delete the guess, which is a decision-12-shaped win: no new construct, one ambiguity gone.

- **Rejected — bare reference sigils** (`#id`, `@id`, `@#id`), on two independent counts: rendering one requires generating its own text, which is decision 15 through a smaller door; and bare `#id` is prose-unsafe (`#general`, `#nowplaying`) and would make `#word` text at line start but syntax mid-line. No digit-guard rule is needed as a result.

- **The validator stays single-document.** Duplicate ids and dangling same-document fragments are lint; a target with a path or a host is a URL, and URLs are not validated — an `https://` link can 404 unchecked, so a cross-document fragment gets the same treatment. This avoids the tooling's first multi-file commitment.
