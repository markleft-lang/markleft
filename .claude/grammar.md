# Grammar sketch — the running rule list

*Working scaffolding, not part of the standard and not normative. Started 2026-08-07. Accumulates as decisions land.*

## What this file is for

A serial list of every rule that gives a character operational meaning, written in the order a parser would apply it, each with an estimate of how often that meaning collides with ordinary writing.

It does three jobs:

- **It shows the collision surface as one list.** Invariant 1 promises that the set of structurally meaningful positions "fits on one page". This is that page. If it stops fitting, that is the invariant failing, visibly, before anyone has to argue about it.

- **It catches collisions between decisions.** A new decision is checked against this list before it is recorded. Section "Findings" below is what fell out of the first pass, and it is not empty.

- **It seeds the Phase 1 grammar.** Every rule here becomes one or more BNF productions. This file is deliberately *looser* than that grammar will be: it is prose plus a form sketch, and it is allowed to say "undecided".

**It is not a specification.** Where this file and `definition.md` disagree, the definition wins and this file is the bug. Where this file says something the definition does not say at all, that is a gap in the definition, and it belongs in `.claude/decisions.md` before it belongs in either.

## How to use it

Rule identifiers are **serial and permanent**. R1 is R1 forever. A rule that is removed is struck and kept, never renumbered, so that a note citing R14 still means what it meant. New rules go at the end of their layer with the next free number, regardless of where they belong logically; the *application order* is given by the "Order" line, not by the number.

Each rule carries the same seven fields:

- **Form** — the shape, sketched. Precision is Phase 1's job.
- **Range** — limits, counts, ceilings.
- **Order** — where it is tried relative to other rules in its layer.
- **Decidable from** — what the parser must see to apply it. Anything other than "this line and the open containers" is an invariant 4 problem.
- **Reserves** — exactly what this rule takes away from plain text, and in what position.
- **Collision** — realistic writing that trips it, and the severity below.
- **Escape** — what an author writes to get the literal text instead.

### Severity scale

| # | Name | Meaning |
|---|------|---------|
| 0 | none | the character has no meaning in this position, guaranteed |
| 1 | negligible | requires a spelling essentially nobody produces by accident |
| 2 | occasional | shows up in technical prose from time to time |
| 3 | frequent | shows up in ordinary prose regularly |
| 4 | hazard | silently changes meaning in common writing, with no visible cue |

**The goal is zero 4s and as few 3s as the language can manage.** A proposed feature that introduces a 3 must pay for it; a 4 is rejected on invariant 1 alone.

---

## Layer 0 — The document

### R1 — Plain text is plain text

- **Form:** anything that matches no other rule.
- **Range:** the whole language.
- **Order:** last, always, at every layer.
- **Decidable from:** nothing — it is the fallback.
- **Reserves:** nothing.
- **Collision:** none, by definition. **Severity 0.**
- **Escape:** not applicable.

This is the prime rule and every other rule in this file is an exception carved out of it. The list below is therefore an exhaustive inventory of what plain text has lost.

### R2 — The document is Unicode code points, encoded UTF-8

- **Form:** a byte stream decoded as UTF-8, then treated as a sequence of code points.
- **Range:** U+0000 to U+10FFFF. Every code point is text.
- **Order:** before all parsing; decoding is a separate, prior step.
- **Decidable from:** the byte stream.
- **Reserves:** nothing. No code point is syntax by virtue of being non-ASCII.
- **Collision:** none. **Severity 0.**
- **Escape:** not applicable.

A leading U+FEFF is not document text; elsewhere it is ordinary text. Content is never normalized, so two strings match only when they are the same code points.

### R3 — Lines

- **Form:** code points up to the next line ending or the end of the document.
- **Range:** a line ending is LF, CR, or CRLF. The last line need not have one.
- **Order:** before block parsing.
- **Decidable from:** the code-point sequence.
- **Reserves:** the line ending, as the unit of block structure.
- **Collision:** none. **Severity 0.**
- **Escape:** `\` before a line ending makes it a hard break (R18); nothing makes it disappear.

### R4 — A column is a count of code points

- **Form:** the first code point of a line is column 1.
- **Range:** all rules that measure indentation.
- **Order:** not applicable — a definition, not a match.
- **Decidable from:** the line.
- **Reserves:** nothing.
- **Collision:** none, but note the consequence — wide characters and multi-code-point graphemes align by count, not by apparent width, so a visually ragged list can be structurally correct and a visually aligned one can be wrong. **Severity 1**, and it lands on CJK and emoji authors rather than evenly.
- **Escape:** not applicable.

### R5 — Whitespace carries no structural meaning of its own

- **Form:** a tab in a structural position is a lint warning, not structure. Trailing whitespace is insignificant.
- **Range:** every block rule that measures a column.
- **Order:** not applicable.
- **Decidable from:** the line.
- **Reserves:** the space, as a separator inside other rules (a heading's space, a list marker's space).
- **Collision:** none in prose; the collision is with *editors* that insert tabs. **Severity 1.**
- **Escape:** none needed — nothing is lost, only warned about.

---

## Layer 1 — Blocks

Applied per line, in the order given. Container rules (R7, R8, R9) consume a prefix and then the remaining rules are applied to what is left of the line.

### R6 — Blank line

- **Form:** a line with no code points, or only spaces.
- **Range:** any position.
- **Order:** 1.
- **Decidable from:** the line.
- **Reserves:** the empty line, as the paragraph separator and the list-looseness signal.
- **Collision:** none. **Severity 0.**
- **Escape:** not applicable.

### R7 — Block quote marker

- **Form:** `>` at the start of the line's remaining content, optionally followed by one space.
- **Range:** nests without limit.
- **Order:** 2, as a container prefix.
- **Decidable from:** this line.
- **Reserves:** `>` in first position.
- **Collision:** a line of prose beginning with a mathematical `>`, or quoted email text that *is* meant as a quote. **Severity 1.**
- **Escape:** `\>`.

### R8 — Bullet list item

- **Form:** `-`, then one space, then content.
- **Range:** `-` is the only bullet marker. A list ends only where a block that is not a list item begins; blank lines never split one.
- **Order:** 3, as a container prefix. Loses to R13 when the line is also a thematic break.
- **Decidable from:** this line and the open containers.
- **Reserves:** `-` in first position when followed by a space.
- **Collision:** **the largest in the language.** A paragraph that opens with a hyphen used as a dash, a minus sign, or dialogue punctuation — `- 5 degrees below`, `- Bonjour, dit-il` — becomes a list item. **Severity 3.** Inherited from Markdown rather than introduced here, and every Markdown writer has already been trained around it, which is the only reason it is tolerable.
- **Escape:** `\-`.

*Revised 2026-08-08.* `*` and `+` were bullet markers and are not any more. They existed so that switching marker could separate two touching lists, so removing them left the "one marker per list" rule with nothing to govern, and that rule is deleted rather than reworded. **`+` moves to the free column of the reserved-position table** — it now has no job anywhere in the grammar, which matters because a diff pasted into prose outside a fence carries `+` on every added line. `*` keeps its other jobs (R13, R21, R22) but loses the precedence contest with the thematic break.

### R9 — Ordered list item

- **Form:** one or more digits, then `.`, then one space, then content.
- **Range:** digit count capped (CommonMark uses 9); the list starts at the first marker's value and renumbers from there. Same list-ending rule as R8.
- **Order:** 4, as a container prefix.
- **Decidable from:** this line and the open containers.
- **Reserves:** a digit run in first position when followed by `.` and a space.
- **Collision:** a paragraph opening with a year or a figure — `1984. That was the year…`, `2026. The licence question is open.` — becomes an ordered list beginning at 1984. **Severity 2**, and it would drop to 1 under the paragraph-interruption rule in Finding 1.
- **Escape:** `\.` after the digits, or `1\. `.

*Revised 2026-08-08.* `1)` was an ordered marker and is not any more. It frees nothing — `)` still closes a link destination under R24 — and goes purely for symmetry with R8: one bullet marker, one ordered marker, list syntax stated in two lines.

**Alphabetic and Roman markers (`a.`, `A.`, `i.`) were proposed and declined**, and the collision is why. `A. Smith argues that…` and `J. R. R. Tolkien wrote…` are initials and author names, ordinary in citation prose, so letters would put a **severity 3** collision at the start of a line — a form that occurs naturally in writing, promoted to structure. They also carry an `i.`-is-letter-or-Roman ambiguity, no rule for what follows `z.`, and, decisively, a numbering *style* written into the source, which is what decision 9 removed classes for. Full argument in the decision record; recorded here so the severity is visible next to the rule that would have carried it.

### R10 — Content column and continuation

- **Form:** an item's content column is where its content begins. Every continuation line is indented to exactly that column.
- **Range:** all list items.
- **Order:** applies while a list container is open.
- **Decidable from:** this line and the open containers.
- **Reserves:** leading spaces, as item structure.
- **Collision:** none in prose. The collision is with *habit* — Markdown's lazy continuation is removed, so an under-indented second line leaves the item instead of joining it. **Severity 2** against fluent Markdown authors, 0 against prose.
- **Escape:** not applicable; indent correctly.

### R11 — Verbatim fence

- **Form:** open with a run of three or more backticks at the start of the line's remaining content, optionally followed by a decorator list (R20). Close with a run of at least as many backticks, alone on a line.
- **Range:** content is verbatim — no escape, no inline rule, no nested decorator. An unclosed fence closes at the end of its container.
- **Order:** 5. Wins over everything except the container prefixes.
- **Decidable from:** this line and the open containers.
- **Reserves:** a run of three or more backticks in first position.
- **Collision:** essentially none — three consecutive backticks do not occur in prose. **Severity 0.**
- **Escape:** `` \` `` on the first backtick.

*Undecided:* whether `~~~` is also a fence. See Finding 4.

### R12 — ATX heading

- **Form:** a run of one to six `#` at the start of the line's remaining content, then exactly one space, then non-empty text.
- **Range:** levels 1 to 6. No closing run, no indentation, no empty heading, no underlined form.
- **Order:** 6.
- **Decidable from:** this line and the open containers.
- **Reserves:** `#` in first position, in a run of one to six, followed by a space.
- **Collision:** a line opening with a number sign meaning "number" and a space — `# 5 is my favourite`. `#hashtag`, `#5`, `#include` and `#######` are all text, because the required space is missing or the run is too long. **Severity 2.**
- **Escape:** `\#`.

The required space is doing almost all the work here, and it is the cleanest example in the language of prose-safety falling out of the form rather than being carved out by an exception.

### R13 — Thematic break

- **Form:** three or more `-`, `_`, or `*`, alone on a line, spaces permitted between them.
- **Range:** any position.
- **Order:** 7, but ahead of R8 when a line satisfies both (`- - -`). Since 2026-08-08 the `***` case no longer arises, `*` having stopped being a bullet marker.
- **Decidable from:** this line.
- **Reserves:** a line consisting only of those characters.
- **Collision:** an ASCII horizontal rule written by hand in a plain-text document — which is what the construct means, so this is a match rather than a collision. **Severity 1.**
- **Escape:** `\-` on the first character.

Worth noting as a dividend of decision 4: with setext headings removed, `---` under a line of text is unambiguously a thematic break. That ambiguity was one of the worst in CommonMark and it is gone by subtraction rather than by rule.

*Undecided:* whether the `___` spelling survives decision 2's removal of underscore. See Finding 5.

### R14 — Table

- **Form:** any line beginning with `|` opens a table. Cells are separated by `|`; a line beginning with `|` and carrying content holds inline cells, and a `|` alone on a line opens a **block cell** whose content is every following line up to the next line beginning with `|`. Rows form by counting cells to the column count.
- **Range:** the column count comes from the first row, rule row or not; a single-column table needs none. **A rule-row cell is three or more `-`, optionally carrying `:` at one end or both** — leading colon left, trailing colon right, both centred, bare run no preference — and a rule row is recognized only as the table's first or second row. Its position says whether there is a header: first line, no header; second row, the row above was the header; absent, neither. **The table ends at a blank line immediately following a line beginning with `|`**, or at the end of its container. The closing mark, a `|` alone on a line before a blank line, opens no cell. Leading and trailing pipes are mandatory on structure lines.
- **Order:** 8.
- **Decidable from:** this line and the open containers.
- **Reserves:** `|` in first position, and `|` as a cell separator inside a table block.
- **Collision:** a line of prose or code beginning with a pipe — BNF alternation, a shell pipeline, `|x| < 5` — is now a one-cell table rather than falling back to a paragraph. **Severity 2**, raised from 1 by the 2026-08-08 rewrite. All three cases are technical rather than everyday, all three belong in a verbatim span anyway, and the failure is visible rather than silent: the text survives and acquires a border. It would drop back to 1 under the paragraph-interruption rule in Finding 1, which this rule therefore makes more valuable.
- **Escape:** `\|`.

*Rewritten 2026-08-08, and this rule is where the design paid for itself twice.* **Finding 2 is closed** — see below — because a table now opens on its own first line like every other block, so nothing in the language looks ahead. And the **leading pipe becomes mandatory by construction**, closing a prose-safety hole GFM leaves open, where an optional leading pipe makes `a | b` in running prose a valid header row.

*Note on the rule row, recorded because documenting the alignment colons is what surfaced it.* GFM accepts a **single** hyphen in a rule-row cell, and gets away with it because its delimiter row can only ever be a table's second line. Once a rule row may also come first, `| - | - |` opening a headerless table parses as structure and **those cells vanish** — a **severity 4** on the scale above, the only class this language treats as disqualifying. Two changes remove it: a three-hyphen minimum, matching R11 and R13, and recognizing a rule row only as the table's first or second row. The residual exposure is a cell whose genuine content is a dash run, in one of those two positions, which needs `\---`. **Severity 1.**

*Note on the terminator, recorded because the first formulation was wrong.* Keying the end of a table on a line **ending** with `|` — the character sequence `|\n\n` — fails on a block cell whose paragraph ends with a pipe: `Write the alternation as a |` would close the table and drop the rest of the cell into the document. Keying on the **first** character adds no hazard the cell rule had not already created, since a content line beginning with `|` already needs `\|`.

### R15 — Link reference definition

- **Form:** `[label]: destination` optionally followed by a title, alone in its own block.
- **Range:** labels match by code-point identity (R2); the definition may appear anywhere in the document.
- **Order:** 9.
- **Decidable from:** this line, plus optionally the next for a title on its own line.
- **Reserves:** a line beginning with `[` when the bracket closes and is followed by `:`.
- **Collision:** a line of prose opening with a bracketed citation followed by a colon — `[Smith]: the argument runs…`. **Severity 1.**
- **Escape:** `\[`.

### R16 — Paragraph

- **Form:** anything else. Inline content (Layer 2) is parsed inside it.
- **Range:** until a blank line or the end of the container.
- **Order:** last.
- **Decidable from:** nothing — the fallback.
- **Reserves:** nothing. **Severity 0.**
- **Escape:** not applicable.

---

## Layer 2 — Inlines

Applied within any block that takes inline content. Verbatim content (R11, R19) is never reached by any rule in this layer.

### R17 — Backslash escape

- **Form:** `\` followed by any single code point yields that code point.
- **Range:** every character, with no exception list. A `\` that is the last code point of the document is a literal backslash.
- **Order:** 1. Before everything, so an escape can defeat any rule below.
- **Decidable from:** two code points.
- **Reserves:** `\` everywhere.
- **Collision:** **the largest inline collision, and the only one this project created rather than inherited.** Under CommonMark a backslash before a letter survives as a literal backslash, so `C:\Users\names` and `\n` and `\d+` are safe in running prose. Under R17 the backslash is consumed and the text silently changes. **Severity 3.**
- **Escape:** `\\`, or — better — write paths and regular expressions in a verbatim span, which is where they belong anyway.

See Finding 3 for the mitigation this argues for.

### R18 — Hard line break

- **Form:** `\` immediately before a line ending.
- **Range:** within a paragraph. It is the only hard break; trailing spaces are not one.
- **Order:** 2, as a special case of R17.
- **Decidable from:** two code points.
- **Reserves:** `\` at end of line.
- **Collision:** a line of prose ending in a backslash — a shell continuation quoted inline. **Severity 1.**
- **Escape:** `\\` at end of line.

### R19 — Verbatim span

- **Form:** a run of one or more backticks, content, then a run of the same length.
- **Range:** content is verbatim; no rule below applies inside it.
- **Order:** 3. Wins over every construct below, which is what makes it the universal safe harbour.
- **Decidable from:** the run length and the search for its match within the block.
- **Reserves:** the backtick, everywhere.
- **Collision:** a backtick in prose — a grave accent typed deliberately, or an apostrophe mistyped as `` ` `` or `´`. **Severity 1**, offset by a real accessibility cost on non-US keyboards that is answered with tooling rather than syntax.
- **Escape:** `` \` ``.

*Undecided:* whether CommonMark's strip-one-leading-and-trailing-space rule is inherited. It has an exception clause, so invariant 2 argues against.

### R20 — Decorator list

- **Form:** a space-separated token list, in exactly two shapes and at most one of each: a bare word (the format label) and `#identifier` (an anchor). Written after a fence's opening backticks, or in braces immediately after a verbatim span's closing run.
- **Range:** tokens exclude whitespace, control characters, and `{`, `}`, `(`, `)`. Nothing else is excluded. The first character selects the shape.
- **Order:** 4, and **only** in those two positions.
- **Decidable from:** the preceding backtick run.
- **Reserves:** `{` immediately after a closing backtick run.
- **Collision:** none in prose, because the position requires a preceding verbatim span. **Severity 0.**
- **Escape:** not needed; put a space before the brace.

No word is ever reserved, so no decorator token can affect the parse. That is what keeps this rule's collision surface at zero permanently rather than at zero for now.

**The braces delimit the list; they are not part of any token.** A fence needs no delimiter, because the rest of the line is the list, so the fence form carries none — ```` ```rust #id ````, never ```` ```rust {#id} ````. Bracing the fence form would nest inside the inline one, `` `x`{rust {#id}} ``. The same `#identifier` therefore appears bare in both list positions and braced only as a standalone anchor (R23), where the braces are that construct's own delimiter. *(Recorded because the asymmetry reads as an inconsistency until the delimiter argument is stated, and it has already caught one reader.)*

### R21 — Emphasis and strong emphasis

- **Form:** `*text*` and `**text**`. An opening delimiter is immediately followed by a non-whitespace character; a closing delimiter is immediately preceded by one.
- **Range:** underscore is never emphasis. Nesting is permitted.
- **Order:** 5.
- **Decidable from:** the delimiter and its immediate neighbours, plus the search for a match within the block.
- **Reserves:** `*` when it flanks non-whitespace.
- **Collision:** `5 * 3 = 15` is safe, because the opener is followed by a space. `*.txt and *.md` is safe, because the second `*` is preceded by a space and cannot close. **But `2*3*4` is not safe** — both delimiters flank non-whitespace, so the middle becomes emphasis. Dense arithmetic and C pointer declarations (`int *p, *q`) are the realistic cases. **Severity 2**, and shared with CommonMark and djot rather than introduced here.
- **Escape:** `\*`.

### R22 — Bracketed emphasis

- **Form:** `{*text*}`, for emphasis inside a word.
- **Range:** the braces are the delimiters, so no flanking condition is consulted.
- **Order:** 6.
- **Decidable from:** `{` followed by `*`.
- **Reserves:** `{` when immediately followed by `*`.
- **Collision:** a brace immediately followed by an asterisk in prose or quoted code — rare enough to be theoretical. **Severity 1.**
- **Escape:** `\{`.

### R23 — Anchor

- **Form:** `{#identifier}`, valid anywhere inline content is valid. Marks a point, not a construct.
- **Range:** identifier character set as R20. Whitespace on either side is optional and insignificant.
- **Order:** 7.
- **Decidable from:** `{` followed by `#`.
- **Reserves:** `{` when immediately followed by `#`.
- **Collision:** template syntax quoted in prose outside a verbatim span — Svelte and Handlebars `{#if}`, `{#each}`. **Severity 2**, concentrated entirely on one audience, and already recorded as a known cost of decision 17.
- **Escape:** `\{`.

### R24 — Inline link

- **Form:** `[text](destination)`, optionally with a title.
- **Range:** the destination may be a URL with a fragment, which is how all cross-referencing works.
- **Order:** 8.
- **Decidable from:** the bracket pair followed immediately by a parenthesis pair.
- **Reserves:** `[` … `]` when immediately followed by `(` … `)`.
- **Collision:** a bracketed aside immediately followed by a parenthetical — `[sic] (or so he claimed)`. The space saves it; without the space it would link. **Severity 1.**
- **Escape:** `\[`.

### R25 — Reference link

- **Form:** `[text][label]`, `[text][]`, or the shortcut `[text]`, each resolved against a definition from R15.
- **Range:** matching by code-point identity.
- **Order:** 9.
- **Decidable from:** the brackets, plus the definition table for the whole document.
- **Reserves:** `[` … `]` in general, conditionally on a definition existing somewhere in the document.
- **Collision:** the shortcut form is the problem. `[sic]`, `[1]`, `[citation needed]`, `[TODO]` are ordinary prose, and any of them becomes a link if a definition with a matching label happens to exist anywhere in the same document. The trigger is non-local and invisible at the point of use. **Severity 2**, rising with document length.
- **Escape:** `\[`.

See Finding 6. This is the one inherited construct whose behaviour depends on text arbitrarily far away.

### R26 — Autolink

- **Form:** `<scheme:rest>` or an email address in angle brackets.
- **Range:** requires a recognized scheme.
- **Order:** 10.
- **Decidable from:** the bracket pair and its content.
- **Reserves:** `<` when it opens a URI.
- **Collision:** `a < b`, `Vec<T>`, `<div>` in quoted prose — none of which carry a scheme, so all are text. **Severity 1**, and *lower here than in Markdown*, because decision 7 removed raw inline HTML: `<div>` is now ordinary text rather than a tag.
- **Escape:** `\<`.

### R27 — Image

- **Form:** `![alt](src)`.
- **Range:** alt text optional; empty alt legal, missing alt is lint.
- **Order:** 11, before R24, since `!` must bind to the bracket that follows it.
- **Decidable from:** `!` immediately followed by a link form.
- **Reserves:** `!` immediately before `[`.
- **Collision:** an exclamation mark immediately before a bracket — `Wow![see below]`. Requires no space, which is unusual in real writing. **Severity 1.**
- **Escape:** `\!`.

### R28 — Text

- **Form:** everything else.
- **Order:** last.
- **Collision:** none. **Severity 0.**

---

## The reserved-position table

Every ASCII punctuation character, and what it costs plain text. This is the one-page statement invariant 1 promises.

| Character | At the start of a line | Inside a line | Worst severity |
|---|---|---|---|
| `` ` `` | fence, run of 3+ (R11) | verbatim span, any run (R19) | 1 |
| `#` | heading, run of 1–6 then a space (R12) | free, except after `{` (R23) | 2 |
| `-` | bullet marker, then a space (R8); thematic break (R13) | free | 3 |
| `*` | thematic break (R13) | emphasis when flanking non-whitespace (R21) | 2 |
| digits | ordered marker, then `.` and a space (R9) | free | 2 |
| `+` | **free** | **free** | **0** |
| `>` | block quote (R7) | free | 1 |
| `\|` | opens a table (R14) | cell separator inside a table only | 2 |
| `\` | escape (R17) | escape (R17); hard break at end of line (R18) | 3 |
| `[` `]` | link reference definition (R15) | link (R24, R25) | 2 |
| `(` `)` | free | link destination, only after `]` (R24) | 1 |
| `!` | free | image, only immediately before `[` (R27) | 1 |
| `{` `}` | free | brace group, only before `#` or `*`, or after a backtick run (R20, R22, R23) | 2 |
| `<` `>` | `>` is a block quote (R7) | autolink, only around a scheme (R26) | 1 |
| `_` | thematic break, `___` only (R13) | **free** | 1 |
| `$` | **free** | **free** | **0** |
| `~` | **free** | **free** | **0** |
| `^` `%` `@` `:` `;` `?` `/` `=` `&` `"` `'` `,` `.` | **free** | **free** | **0** |

Fourteen ASCII punctuation characters have no meaning anywhere in the language, and three more — `$`, `_`, and `+` — were deliberately bought back. That is the concrete form of invariant 1, and it is the number to watch: **any future decision that moves a character out of the free column is spending the language's main asset, and any decision that moves one into it is the language's main dividend.**

*Changed 2026-08-08:* `+` left the reserved column entirely when it stopped being a bullet marker, and `*` dropped from severity 3 to 2 by losing the same job. No character moved the other way.

---

## Findings from the first pass

Six things this list surfaced that the decision record does not settle. Each is a candidate for the record; none is decided here.

### Finding 1 — "No block interrupts a paragraph" would cut the collision surface substantially, and is not on the record

`.claude/landscape.md` lists "block elements can't interrupt paragraphs" among djot's design principles that we share. It appears nowhere in `.claude/decisions.md`, and `definition.md` does not state it.

If adopted, a line inside an open paragraph is text no matter what it looks like, and R8, R9, R12, R13, and R14 all stop firing mid-paragraph. That drops R9 from severity 2 to 1, softens R12, and removes the whole class of "my paragraph turned into a list because it started with a year".

The cost is a real surprise in the other direction: a list written immediately under a line of prose, with no blank line between, stops being a list. That is a genuine break with Markdown habit and belongs in the deltas appendix if it is taken.

**Worth deciding explicitly, either way**, because the current state is that a principle we claim to share is not written down anywhere binding.

### Finding 2 — CLOSED 2026-08-08 — pipe tables needed one line of lookahead

**Resolved by design change, not by amendment.** Any line beginning with `|` now opens a table, so a table is decidable from its own first line like every other block and no rule in the language looks ahead. Invariant 4 stands as written, and decision 4 keeps both of its arguments against setext headings.

Kept here because the shape of the resolution is worth reusing. The finding was that a header row was indistinguishable from a paragraph until the rule row arrived, which is exactly the property setext headings were removed for — a real conflict between decisions 4 and 8. The obvious fix was constitutional: narrow invariant 4 to a uniform one-line confirmation window, at the cost of one of decision 4's two arguments. The better fix changed the construct so the conflict could not arise. **When a finding here points at an invariant, try moving the construct before moving the invariant.**

See the amendment of 2026-08-08 in `.claude/decisions.md`.

### Finding 3 — Decision 3 is the only decision that *increases* the collision surface

Every other binding decision takes a character out of syntax or narrows a form. Uniform escaping does the opposite: under CommonMark a backslash before a letter survives, so `C:\Users`, `\n`, and `\d+` are safe in running prose; under R17 the backslash is consumed and the text changes silently.

This is not an argument against decision 3, whose "no exception list" guarantee is worth more. It is an argument that the decision needs a mitigation attached, and there is an obvious one: **a validator warning on a backslash immediately followed by an alphanumeric character**, asking whether a literal backslash was meant. That catches Windows paths, regular expressions, and TeX pasted into prose, at the point of failure, without touching the grammar. It is the same shape as the backtick-lookalike diagnostic already proposed in `notes/backtick-verbatim-challenge.md`.

Belongs in the Phase 3 validator list.

### Finding 4 — Tilde fences are undecided

CommonMark admits `~~~` as an alternative fence. The decision record never mentions it, `deltas.md` has no row for it, and every statement of decision 9's decorator grammar is written in terms of backticks.

Removing them is a new delta and is what the one-way-to-do-it discipline argues for. Keeping them costs a second spelling of one construct. Either way it should be stated, because right now an implementer would have to guess.

Note the knock-on: `~` is currently in the free column of the table above. Admitting tilde fences takes it out.

### Finding 5 — Strikethrough and HTML entity references are absent by accident, not by decision

Two GFM and CommonMark constructs are simply not mentioned anywhere in this project:

- **Strikethrough (`~~text~~`)** is GFM, not CommonMark, so decision 12 does not import it and it is absent. Fine — but GFM users will look for it, and "we considered it and declined" reads differently from silence. It is also the second claimant on `~`, alongside Finding 4.

- **HTML entity references (`&amp;`, `&#169;`)** are in CommonMark, so decision 12 arguably *does* import them. They should almost certainly go: an entity reference renders a character the source does not contain, which is decision 15's front half through a very small door, and resolving one requires an HTML entity table, which is a dependency on a serialization the language claims independence from. Removing them is a delta and is not in `deltas.md`.

Both need a line in the record. The entity question is the substantive one.

### Finding 6 — Shortcut reference links are the only construct with action at a distance

`[sic]`, `[1]`, `[TODO]`, and `[citation needed]` are ordinary prose. Under R25's shortcut form, any of them becomes a link if a definition with a matching label exists anywhere in the same document — including one added months later, in an unrelated edit, by someone who never saw the paragraph they changed.

Nothing else in the language behaves this way. Every other rule is decidable from a few code points and their immediate neighbours; this one is decidable only against a document-wide table, and it fails silently when it fires.

Dropping the shortcut form — keeping `[text][label]` and `[text][]`, which are explicit — would be a delta that costs almost nothing and removes the last non-local construct. Worth putting to the record as a prose-safety improvement rather than as a simplification.

---

## Maintenance

Add a rule when a decision creates one, in the same session that records the decision. Update the reserved-position table in the same edit — it is the summary that gets read, and a stale summary is worse than none.

When a finding here is settled, replace it with a one-line pointer to the amendment in `.claude/decisions.md`. Findings accumulate; resolutions live in the record.
