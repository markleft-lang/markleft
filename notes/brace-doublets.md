# Memo: one shape for every inline construct

*Non-normative. Status: challenge raised 2026-08-09, unresolved. Proposes replacing the inline layer of the language with a single rule, and records the accounting so a later session finds the analysis rather than rebuilding it. Touches decisions 2, 3, 11, 12, 13, 16, and 17, and the self-hosting ladder in `CLAUDE.md`. Nothing here is recorded in `.claude/decisions.md` or `.claude/grammar.md` yet.*

## The proposal in one line

**A sigil means nothing unless the very next character is `{`.**

That sentence is not new. It is decision 18, written for `^` and `_`, and the proposal is that it was never a rule about superscripts — it was the language's organizing principle, discovered on two characters and stopped there.

Applied to the whole inline layer:

| Sigil | Construct | Written |
|---|---|---|
| `*` | emphasis | `*{em}` |
| `**` | strong | `**{strong}` |
| `***` | both | `***{both}` |
| `^` | superscript | `10^{23}` |
| `_` | subscript | `H_{2}O` |
| `#` | anchor | `#{section-3}` |
| `@` | link | `@{https://example.org}`, `@[Tessier et al.]{#tessier}` |
| `!` | image | `![a kangaroo in Cairns]{assets/kangaroo.png}` |
| `\` | escape range | `a \{|} b` |
| `` ` `` | format label | `` `x=y`{math} `` |

Every opener is a fixed two-character literal. Nothing else in a line is operative. Block structure is untouched: `#` and a space still opens a heading, `-` and a space still opens a bullet, `` ``` `` still opens a fence, `>` still quotes, `|` still opens a table.

## Three findings

The case does not rest on consistency. It rests on three things that are true of the language as it stands today.

### 1. Decision 3 is less prose-safe than CommonMark

Decision 3 says a backslash escapes **any** character, with no exception list, and R17 implements it. So today:

```
C:\Users\frederic
```

renders as `C:Usersfrederic`. CommonMark escapes only ASCII punctuation, so `\U` stays `\U` and the path survives.

This is a Windows path, a regular expression, a `\n` quoted in running prose. It is ordinary technical writing, silently altered, with no visible cue — severity 4 on the grammar file's own scale, which is the band invariant 1 does not admit at all. The rule reached it by being *more* principled than CommonMark, which is exactly why nobody has noticed: the rule reads perfectly.

Under the doublet rule `\` joins every other sigil — inert unless followed by `{` — and the defect closes without an exception list, because the repair is the same rule everything else follows.

### 2. Decision 11 does not protect emphasis

Decision 11 keeps Markdown muscle memory *where Markdown was not broken*. Emphasis is the most broken construct CommonMark has: left- and right-flanking classification, the rule of 3, `*` and `_` behaving differently inside a word, and a `***` nesting corner nobody predicts correctly. The construct decision 11 was written to protect is the link. Emphasis fails its test on the merits, and it always did.

### 3. Braced emphasis removes invariant 1's last heuristic

Invariant 1 promises prose-safety **by construction, never by heuristic**. One construct has never met that: `5 * 3 = 15` is safe today because of flanking rules, which are a heuristic by any reading. R21 is the last place in the language where an operative meaning is decided by looking at what surrounds a character rather than at the character itself.

`*{…}` deletes it. After this change invariant 1 holds without residue, and no other available change does that.

## The ledger

`.claude/grammar.md` names the number to watch: *any future decision that moves a character out of the free column is spending the language's main asset, and any decision that moves one into it is the language's main dividend.*

**Spent: one character.** `@` leaves the free column, at severity 1, in one position only — immediately before `{` or `[`.

**Earned:**

| Character | Now | Under the proposal | |
|---|---|---|---|
| `\` | 3 | 1 | the last severity-3 meaning inside a line |
| `*` | 2 | 1 | flanking gone; operative only before `{` |
| `[` `]` | 2 | 1 | operative only immediately after `@` or `!` |
| `{` `}` | 2 | 1 | operative only immediately after a sigil |
| `(` `)` | 1 | **0** | no construct uses parentheses |
| `<` | 1 | **0** | if autolinks go with reference links (open question 3) |
| `@` | **0** | 1 | the cost |

The rebuilt table is mocked in full at `.claude/grammar-new.md`, alongside the doublet inventory and the severity-1 collisions this introduces. Two consequences worth reading off it directly:

**`-` becomes the only severity-3 character in the language.** Today there are two.

**The line-start column contains exactly the block markers and nothing else.** It currently holds `` ` ``, `#`, `-`, digits, `>`, `|`, `\`, and `[`. Under the proposal `\` and `[` leave it, and what remains is the six characters that open blocks. Class 1 of the language stated as a table.

## The language in three sentences

1. **Blocks are decided by position in the line** — the start opens one, the end breaks one.
2. **Inlines are decided by a sigil immediately followed by `{`.**
3. **Verbatim is delimited by matching backtick runs.**

Sentence 3 is an exception, and it explains itself: verbatim content is not interpreted, so its delimiter cannot be balance-counted — closing the span would mean reading the content. Run-length matching is the only escape that works without looking inside. That is why verbatim keeps the paired form and nothing else does, and one sentence is the bar invariant 2 sets.

The corollary settles a question the proposal raises on its own: `\{…}` and backtick verbatim are not redundant. The escape range is literal but brace-balanced; verbatim is literal, period.

## Mechanics

**Closing.** A brace construct closes on a run of exactly the opening length — the same sentence decision 5 already uses for fences. Runs are maximal, so `{{{` is one run of three.

**Nesting.** Single braces work whenever the content is balanced: `\{a {b} c}` closes at the final `}`, with no lengthening. Lengthening is the escape hatch for the case that actually fails — `*{the } character}` breaks, and `*{{the } character}}` does not, because a lone `}` is a run of one and cannot close a run of two.

This makes the language's escape story one sentence: **when a delimiter appears in your content, lengthen the delimiter.** Fences, verbatim spans, and brace constructs, identically. Nothing else.

**Run length caps itself, with precedent.** Emphasis runs of 1, 2, and 3 are defined; a run of 4 or more is not a construct, so the run and the brace are both text. That is decision 4's shape — one to six `#`, and seven is text.

**Two brace flavours, one syntax.** Literal content (`\{…}`, `@{url}`, `#{id}`) counts brace depth. Inline content (`*{…}`, `^{…}`, `_{…}`) is parsed recursively, so a nested construct is consumed whole and its braces never reach the outer close. Both are linear with no backtracking, both honour the same run-length escape, and neither changes anything an author types. Phase 1 has to say which is which, with a worked example of the corner where they meet.

**The hard break stays exactly as decision 13 has it** — `\` before a line ending, and nothing else. Three reasons, and they converge:

- It is the most cardinal-rule-compliant construct in the language. Decision 19 removed the thematic break for having no plain-text meaning; the hard break has the strongest possible one, since the source line ends exactly where the output line ends. Nothing else in the language is isomorphic to its own rendering.

- Its one realistic collision renders correctly. A shell continuation quoted in unfenced prose — `gcc -o foo \` — becomes a hard break, which is the line break the author already wanted.

- A literal trailing backslash keeps an answer with no new machinery: `the drive root is C:\{\}`.

It is also not the exception to left-marking that it first appears to be. `\` before a line ending is the sigil operating on the line ending — sitting immediately to its left, which is the shape of `\{` exactly. So `\` takes two operands, a brace range and a line ending, and the second is structurally forced rather than chosen: a line ending is the one character that cannot appear inside a brace range, because a brace range is inline. Decision 3 already said as much, in the clause about "explicitly including a line ending."

`\\` was considered and does not earn its place. A trailing `\\` is exactly as invisible as a trailing `\`, so it buys nothing on intentionality, and it puts UNC paths (`\\server\share`) and escaped backslashes in quoted code into the blast radius. `\n` reintroduces the exception list decision 3 exists to avoid.

**`\{}` is a no-op**, and correctly so: an escape range with no content renders nothing. It is not a hard break, and it is what a search-and-replace leaves behind when it empties one.

## What the change costs

**A clean break with Markdown, chosen rather than absorbed.** Under this proposal a Markleft document containing a link or emphasis is not valid CommonMark and does not render as CommonMark. That is the largest divergence the project has proposed, and the argument for taking it deliberately is the one Markdown itself supplies: near-compatibility is what produced flavour drift, and flavour drift is what leaves a writer unable to answer whether their document will survive a move between platforms. A vendor can support Markdown and Markleft side by side; what stops paying is the temptation to stretch one implementation across both.

The two-extension decision (§6) turns out to be the instrument that makes this work — `.markleft` and `.lf` are the discriminator that keeps the two languages from blending. It was decided for other reasons and is load-bearing here.

**Decision 12 changes character.** "Match CommonMark byte-for-byte wherever it is unambiguous and harmless" stops being a design constraint and becomes an observation: the block layer *is* Markdown's, because Markdown's block layer was right, not because compatibility asked for it. The deltas appendix survives with a different job — a translation guide for people who know Markdown, rather than a promise that documents port.

**Self-hosting stages 1 and 2 do not survive.** `CLAUDE.md` stage 1 is "every `.md` file is valid Markleft that renders identically under CommonMark," and stage 2 makes it a CI check. Any document containing a link or emphasis fails it, which is every document in these repositories. The measurement stage 1 was built to produce — the fraction of our own documents requiring `.markleft` — still computes, but its answer becomes known in advance and stops being informative. Stage 3 (`.markleft` sources with generated `.md` companions) has to arrive around Phase 3, or project prose stays CommonMark-flavoured and is honestly labelled as not yet dogfooded.

**New collisions, all small, listed rather than smoothed over.** The doublets introduce a family of severity-1 openers, of which two deserve naming: `#{` is Ruby and Elixir string interpolation, which appears in prose about those languages; `*{` is shell brace expansion after a glob. Both are answered by a verbatim span, which is where such text belongs anyway, and neither is close to the band the removed collisions occupied.

**Typing cost, counted rather than asserted:**

| | CommonMark | Markleft | |
|---|---|---|---|
| emphasis | `*em*` (2) | `*{em}` (3) | +1 |
| strong | `**em**` (4) | `**{em}` (4) | free |
| both | `***em***` (6) | `***{em}` (5) | −1 |

One character, on one of three forms, and the heaviest form gets cheaper.

**Keyboard load.** Braces are AltGr on AZERTY and several other layouts, and this moves every operative inline construct behind that. The counter is the one `notes/backtick-verbatim-challenge.md` already established and accepted: a character's inaccessibility and its prose-safety are the same property, and `(` and `[` are far more common in running prose than `{`. The mitigation is tooling — an editor can close the brace when a non-space character follows a sigil — and the mitigation is partial by construction, because a phone keyboard, a GitHub textarea, and `git commit -m` have no extension. Both halves belong in the cheat sheet.

## What the change buys beyond the invariants

**The last inline lookahead closes.** Decision 8 claimed to close "the last lookahead in the language" with tables, and that was not quite true: bare `[` still requires a scan to `]` and a check for `(`. `@[` decides on character two. The inline layer becomes prefix-decidable in the same sense decision 4 gives blocks.

**CommonMark's hardest algorithm evaporates.** The delimiter stack, flanking classification, and the rule of 3 exist to resolve emphasis. `*{a **{b} c}` is unambiguous by balance alone. This is the largest single simplification available anywhere in the Markdown-like design space.

**Search becomes exact.** Every construct has a fixed literal opener, so `[@!^_*#\\]+\{` finds every operative inline construct in a document, at any nesting depth — the un-regular part is only the *extent*, never the *presence*. Inventory questions are regular and exact; extent questions need a parser, and almost every question tooling asks is an inventory question. Note also which constructs carry opaque content: `@{url}`, `#{id}`, and `` `x`{math} `` cannot contain a nested construct, so `@\{[^{}]*\}` is complete for links rather than an approximation. The constructs a person most often batch-rewrites are the regular ones.

**Syntax highlighting can be correct.** Editor grammars mis-highlight CommonMark emphasis everywhere, because flanking rules are not expressible in the pattern languages those grammars use. A `begin`/`end` grammar tracks brace nesting natively.

**LLM emitters get shorter instructions.** A model has to learn flanking rules and the rule of 3 to emit CommonMark correctly; it has to learn `sigil{content}` to emit Markleft. The Phase 4 system-prompt-sized language description gets materially shorter and more reliable, which is an argument available in 2026 that was not available to Markdown in 2004.

## The name turns out to describe the design

**Every construct's mark sits to the left of that construct's content.** Blocks mark the left of a line; inlines mark the left of a span. The block layer was already left-marked — `#`, `-`, `>`, `|` are all line-start prefixes — so this is not a property of the new inline rule, it is a property of the whole language that the new rule completes.

Stated that way it holds without exception, and the two constructs that look like exceptions are worth working through, because the loose phrasing "everything is marked on the left" needs footnotes the exact one does not.

**Verbatim is not an exception.** A span is an opener plus a scope end, which is what `*{…}` is. Only the closing character differs — a matching backtick run rather than `}` — and the mark is the opening run either way.

**The hard break is not an exception**, for the reason given under Mechanics: `\` sits immediately to the left of the line ending it marks.

**Decorators are, and they are a different thing.** `` `x=y`{math} `` and ```` ```math ```` place the annotation after what it annotates. The mark is still left of its own content — `{math}` is opened by the backtick before it — but the relationship points backwards. That is annotation rather than marking, and it is the one place in the language the arrow reverses. It is also open question 5 from a second direction.

This is a retronym and the prose should say so — the name came first, and it turns out to describe the design. Four readings now stack: *sideways* from markup and markdown, *what is left* after fifteen years of curation, *marks on the left*, and the tagline below, where the name is the verb in the past tense. Only the second is a joke, so the humour budget is untouched.

### The tagline

> **Everything that is not prose leaves a mark.**

Its content is the contrapositive: **no mark, therefore prose.** That is invariant 1 stated from the reader's side rather than the parser's, and the cardinal rule at the same time, without naming either.

It is also not true of the language as it stands. "Leaves a mark" requires that a mark be findable, and emphasis today is decided by flanking — whether `*` is a mark or a character is settled by what surrounds it, which a reader cannot see. So the sentence is an argument for this proposal rather than a description of the language, and it becomes true on the day the proposal lands.

*"Leaves a **clear** mark"* was considered and the adjective went. It adds nothing the sentence does not already carry, and it invites a footnote about the hard break — the one mark a reader cannot see — which the shorter form does not need.

**Markless** was considered and does not survive this change. "Less" reads as austerity, which `CLAUDE.md` explicitly disclaims, and the language now marks more visibly rather than less.

## Open questions

1. **Does `**{…}` exist?** Symmetry says yes and run-length gives it away free. It is near-unused, and deferring it changes no rule.

2. **Does `{*…*}` have a successor?** Decision 2's bracketed emphasis (R22) becomes `*{…}`, which is the same construct with the brace on the other side. Nothing else moves — decision 2 already closed intra-word `*`, which is what R22 exists to reopen.

3. **Do reference links (R15) and autolinks (R26) survive the clean break?** Decision 17 already says references need no new syntax, and `@{https://example.org}` covers the autolink case. Removing both frees `<` outright and takes `[` out of the line-start column. The ledger above assumes they go; the decision has not been made.

4. **Does `@` take `[label]` or something else?** `@[text]{url}` puts the human-facing part in brackets and the machine-facing target in braces, which reads consistently with `` `x`{math} `` — *what you see in front, what it means in the braces.* The cost is that `@` and `!` are the two sigils whose next character is `[` rather than `{`, which is a footnote on the one-line rule.

5. **What does a decorator on a fence look like?** Inline decorators are braced (`` `x`{math} ``); fence decorators are a bare word after the run (`` ```math ``). That asymmetry is deliberate — the fence already delimits, so there is nothing for a brace to scope — but it is the one place the language has two shapes for one vocabulary, and the definition should say why rather than leave it noticed.

## What recording this takes

This memo changes nothing. If the proposal is adopted, the same session lands all of:

- `.claude/decisions.md` — a new decision 20, and amendments to 2, 3, 11, 12, 16, and 17 recorded in §11 rather than edited silently.
- `.claude/grammar.md` — the reserved-position table rebuilt from the ledger above, R17, R18, R21, R22, R23, R24, R25, R27, and R29 rewritten, R15 and R26 struck if question 3 goes that way, and new rules for the escape range and the brace run-length close.
- `definition.md` — §4 and §5, and Appendix D, which gains nothing and loses nothing here.
- `deltas.md` — reframed from migration guide to translation guide.
- `CLAUDE.md` — the self-hosting ladder, the decision summaries, and the "close cousin" framing.
- `.github/profile/README.md` — only if the phase list moves.

The branch is `brace-doublets`, to be merged `--no-ff` so the bubble stays visible and droppable.
