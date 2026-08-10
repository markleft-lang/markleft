# Grammar sketch — the running rule list

*Working scaffolding, not part of the standard and not normative. Started 2026-08-07. Accumulates as decisions land.*

## The reserved-position table

Every ASCII punctuation character, and what it costs plain text. This is the one-page statement invariant 1 promises, and it is first in the file because it is the part you consult while typing — what is safe to write, at a glance, without reading a rule.

Each entry cites the rule that claims it (`R11`, `R30`, …); those rules are the rest of this file, in the order a parser applies them.

**Rows run from the biggest cost to plain text down to none**, so the top of the table is what to watch when writing and the bottom is what is always safe. Within a severity band the original order is kept, which is why a band is not itself ranked.

| Character | At the start of a line | Inside a line | Worst severity |
|---|---|---|---|
| `-` | bullet marker, then a space (R8) | free | 3 |
| `#` | heading, run of 1–6 then a space (R12) | anchor, only before `{` (R23) | 2 |
| digits | ordered marker, then `.` and a space (R9) | free | 2 |
| `\|` | opens a table (R14) | cell separator inside a table only (R14) | 2 |
| `` ` `` | fence, run of 3+ (R11) | verbatim span, any run (R19); format label before `{` (R20) | 1 |
| `>` | block quote, then a space (R7) | free | 1 |
| `\` | free | escape range, only before `{` (R17); hard break at end of line (R18) | 1 |
| `*` | free | emphasis, only before `{`, runs of 1–3 (R21) | 1 |
| `_` | free | subscript, only before `{` (R29) | 1 |
| `^` | free | superscript, only before `{` (R29) | 1 |
| `@` | free | link, only before `{` or `[` (R24) | 1 |
| `!` | free | image, only before `{` or `[` (R27) | 1 |
| `[` `]` | free | link or image text, only immediately after `@` or `!` (R24, R27); state box at the start of a bullet item's content (R31) | 1 |
| `{` `}` | free | scope, only immediately after a sigil (R30) | 1 |
| `~` | free | strikethrough, only before `{` (R32) | 1 |
| `<` | aside marker, then one space, or alone on the line (R33) | free | 1 |
| `(` `)` | **free** | **free** | **0** |
| `+` | **free** | **free** | **0** |
| `$` | **free** | **free** | **0** |
| `%` `:` `;` `?` `/` `=` `&` `"` `'` `,` `.` | **free** | **free** | **0** |

**Fifteen** of the thirty-two ASCII punctuation characters have no meaning anywhere in the language: the eleven in the last row, plus `(`, `)`, `+`, and `$`. That is the concrete form of invariant 1, and it is the number to watch: **any future decision that moves a character out of the free column is spending the language's main asset, and any decision that moves one into it is the language's main dividend.**

Two things are worth reading off the table directly, because neither is stated by any single rule:

**`-` is the only severity-3 character in the language.** Its collision is R8's, it is inherited from Markdown rather than introduced here, and every Markdown writer has already been trained around it.

**The line-start column holds exactly the seven block markers** — `` ` ``, `#`, `-`, digits, `>`, `<`, `|` — and nothing else. That is Layer 1 stated as a table, and it is what makes a block's type decidable from its own first character. *(Six until decision 23 added the aside; the six-marker form of this sentence is quoted as an asset in decisions 20 through 22, and the growth was spent knowingly.)*

*Changed 2026-08-10 — decision 23's same-day amendment.* The `>` row tightens: its space, optional since email, is required — the last optional-space clause in the language. `>=50%`, `>>log`, and `>5ms` at line start become text by construction; severity stays 1, narrower.

*Changed 2026-08-10 — decision 23.* `<` is spent at the start of a line — the aside marker, the block quote mirrored — and stays free inside one. The free count drops to fifteen, and the line-start column grows to seven for the first time since it was frozen at six. `<` had been freed by decision 20 the day before; the fast respend is recorded rather than smoothed over.

*Changed 2026-08-10 — decision 22.* `~` is spent — strikethrough, one position (immediately before `{`), at severity 1. The first character since `@` to leave the free column outright, and the free count drops to sixteen.

*Changed 2026-08-10 — decision 21.* `[` gains one position: the three-character state box (`[ ]` or `[x]`) at the start of a bullet item's content. Severity unchanged at 1 — a narrow, partial re-spend of the position decision 20 eased from 2 to 1.

*Changed 2026-08-09 — decision 20, the largest single revision this table has had.* One character spent (`@`), nine characters across the six other rows eased or freed outright, and the language's last severity-3 inline meaning removed.

| Character | Before | After | What moved |
|---|---|---|---|
| `@` | **0** | 1 | **spent** — link, before `{` or `[` |
| `\` | 3 | 1 | escape narrowed to `\{…}`; R17's collision closed |
| `*` | 2 | 1 | flanking gone; operative only before `{` |
| `[` `]` | 2 | 1 | operative only after `@` or `!`; leaves the line-start column |
| `{` `}` | 2 | 1 | operative only after a sigil, never on its own |
| `(` `)` | 1 | **0** | no construct uses parentheses |
| `<` | 1 | **0** | autolinks struck with R26 |

The `\` row is the one that mattered, and it closed a defect rather than tidying a shape. Under the previous R17 a backslash consumed the character after it with no exception list, so `C:\Users\frederic` rendered as `C:Usersfrederic` — ordinary technical writing altered with no visible cue, which is severity 4 on the scale below and the band invariant 1 does not admit at all. Finding 3 had recorded the problem and proposed a validator warning; decision 20 removed the cause instead.

*Changed 2026-08-08, third revision — the dividend side.* `*` and `_` each got their **line-start column back** when decision 19 removed the thematic break (R13).

*Changed 2026-08-08, second revision — the first entry that spends rather than earns.* `^` left the free column, and `_` gained a meaning inside a line, both to decision 18. Two characters moved out, each into one position only — immediately before `{` — and each landing at severity 1. That rule became the shape decision 20 generalized.

*Changed 2026-08-08:* `+` left the reserved column entirely when it stopped being a bullet marker, and `*` dropped from severity 3 to 2 by losing the same job.

### How to read the severity column

| # | Name | Meaning |
|---|------|---------|
| 0 | none | the character has no meaning in this position, guaranteed |
| 1 | negligible | requires a spelling essentially nobody produces by accident |
| 2 | occasional | shows up in technical prose from time to time |
| 3 | frequent | shows up in ordinary prose regularly |
| 4 | hazard | silently changes meaning in common writing, with no visible cue |

**The goal is zero 4s and as few 3s as the language can manage.** A proposed feature that introduces a 3 has to pay for it; a 4 does not survive invariant 1 on its own.

### The doublet table

Layer 2 in one table. **A sigil means nothing unless the very next character is `{`** — extended by `[` for the link and image forms that carry their own text — so every operative inline construct opens on a sigil run and one deciding code point, and every character in the list below is ordinary text everywhere else.

| Opener | Construct | Written | Brace holds | Rule |
|---|---|---|---|---|
| `*{` | emphasis | `*{em}` | inline content | R21 |
| `**{` | strong | `**{strong}` | inline content | R21 |
| `***{` | both | `***{both}` | inline content | R21 |
| `^{` | superscript | `10^{23}` | inline content | R29 |
| `_{` | subscript | `H_{2}O` | inline content | R29 |
| `~{` | strikethrough | `~{no longer}` | inline content | R32 |
| `#{` | anchor | `#{tessier-2026}` | literal | R23 |
| `@{` | link | `@{https://example.org}` | literal target | R24 |
| `@[` | link with text | `@[Tessier et al.]{#tessier-2026}` | literal target | R24 |
| `!{` | image | `!{assets/kangaroo.png}` | literal target | R27 |
| `![` | image with alt | `![a kangaroo in Cairns]{assets/kangaroo.png}` | literal target | R27 |
| `\{` | escape range | `a \{\|} b` | literal | R17 |
| `` `{ `` | format label | `` `x=y`{math} `` | literal | R20 |

**The brace-holds column is the only distinction that reaches the parser**, and it is R30's business rather than any construct's: inline content is parsed recursively, literal content counts brace depth. Neither changes what an author types.

**Every mark sits to the left of its own content.** Blocks mark the left of a line, inlines mark the left of a span, and `\` before a line ending marks the left of the line ending. The one place the arrow reverses is the decorator, which follows what it annotates — R20, and it is annotation rather than marking.

## What this file is for

A serial list of every rule that gives a character operational meaning, written in the order a parser would apply it, each with an estimate of how often that meaning collides with ordinary writing.

It does three jobs:

- **It shows the collision surface as one list.** Invariant 1 promises that the set of structurally meaningful positions "fits on one page". This is that page. If it stops fitting, that is the invariant failing, visibly, before anyone has to argue about it.

- **It catches collisions between decisions.** A new decision is checked against this list before it is recorded. The "Findings" section below is what falls out of each pass, and it is not empty.

- **It seeds the Phase 1 grammar.** Every rule here becomes one or more BNF productions. This file is deliberately *looser* than that grammar will be: it is prose plus a form sketch, and it can say "undecided".

**It is not a specification.** Where this file and `definition.md` disagree, the definition wins and this file is the bug. Where this file says something the definition does not say at all, that is a gap in the definition, and it belongs in `.claude/decisions.md` before it belongs in either.

## How to use it

Rule identifiers are **serial and permanent**. R1 is R1 forever. A rule that is removed is struck and kept, never renumbered, so that a note citing R14 still means what it meant. New rules go at the end of their layer with the next free number, regardless of where they belong logically; the *application order* is given by the "Order" line, not by the number.

Each rule carries the same seven fields:

- **Form** — the shape, sketched. Precision is Phase 1's job.
- **Range** — limits, counts, ceilings.
- **Order** — where it is tried relative to other rules in its layer.
- **Decidable from** — what the parser must see to apply it. Anything other than "this line and the open containers" is an invariant 4 problem.
- **Reserves** — exactly what this rule takes away from plain text, and in what position.
- **Collision** — realistic writing that trips it, and its severity on the scale at the top of this file.
- **Escape** — what an author writes to get the literal text instead.

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

## Layer 1 — Blocks

Applied per line, in the order given. Container rules (R7, R33, R8, R9) consume a prefix and then the remaining rules are applied to what is left of the line.

**Every block-opening rule in this layer is decided by the first character of the line's remaining content**, and the seven characters that can be that character are `` ` ``, `#`, `-`, a digit, `>`, `<`, and `|`. Nothing else opens a block. The two rules that read no character make no exception to that — a blank line (R6) has no remaining content, and continuation (R10) is measured as indentation before any character is read — and no rule here looks at any line but its own.

### R6 — Blank line

- **Form:** a line with no code points, or only spaces.
- **Range:** any position.
- **Order:** 1.
- **Decidable from:** the line.
- **Reserves:** the empty line, as the paragraph separator and the list-looseness signal.
- **Collision:** none. **Severity 0.**
- **Escape:** not applicable.

### R7 — Block quote marker

- **Form:** `>` at the start of the line's remaining content, then one space, then content — or `>` alone on the line, the quote's blank line.
- **Range:** nests without limit; a nested quote is written `> > text`.
- **Order:** 2, as a container prefix.
- **Decidable from:** this line.
- **Reserves:** `>` in first position when followed by one space, or alone on the line.
- **Collision:** a line of prose opening with a spaced comparison — `> 5 ms was typical` — or pasted email text that *is* meant as a quote. **Severity 1.** `>=50%`, `>>log`, and `>5ms` are all text, the required space missing.
- **Escape:** `\{>} 5 ms`.

*Revised 2026-08-10 — decision 23's same-day amendment.* The space was optional, inherited from email via CommonMark, and it was the last optional-space clause in the language — every other marker already required its space. Required now, for the three reasons the others require theirs: one spelling per construct, the `<`/`>` mirror completed, and the line-start comparisons (`>5ms`, `>=`) becoming text by construction exactly as `<5ms` and `<div>` did under R33. The cost is the old email idiom: `>quoted` and `>>nested` fail the form and are text — loudly, not silently — and the migrator rewrites both mechanically (`>text` → `> text`, `>>` → `> > `).

### R8 — Bullet list item

- **Form:** `-`, then one space, then content.
- **Range:** `-` is the only bullet marker. A list ends only where a block that is not a list item begins; blank lines never split one.
- **Order:** 3, as a container prefix. No exception — the one it carried, losing to R13 on a line that was also a thematic break, left with R13.
- **Decidable from:** this line and the open containers.
- **Reserves:** `-` in first position when followed by a space.
- **Collision:** **the largest in the language.** A paragraph that opens with a hyphen used as a dash, a minus sign, or dialogue punctuation — `- 5 degrees below`, `- Bonjour, dit-il` — becomes a list item. **Severity 3.** Inherited from Markdown rather than introduced here, and every Markdown writer has already been trained around it, which is the only reason it is tolerable.
- **Escape:** `\{-}`.

*Revised 2026-08-08.* `*` and `+` were bullet markers and are not any more. They existed so that switching marker could separate two touching lists, so removing them left the "one marker per list" rule with nothing to govern, and that rule is deleted rather than reworded. **`+` moves to the free column of the reserved-position table** — it has no job anywhere in the grammar, which matters because a diff pasted into prose outside a fence carries `+` on every added line. *(And on 2026-08-08 `*` lost the thematic break too — decision 19 — so it no longer claims the first column at all.)*

### R9 — Ordered list item

- **Form:** one or more digits, then `.`, then one space, then content.
- **Range:** digit count capped (CommonMark uses 9); the list starts at the first marker's value and renumbers from there. Same list-ending rule as R8.
- **Order:** 4, as a container prefix.
- **Decidable from:** this line and the open containers.
- **Reserves:** a digit run in first position when followed by `.` and a space.
- **Collision:** a paragraph opening with a year or a figure — `1984. That was the year…`, `2026. The licence question is open.` — becomes an ordered list beginning at 1984. **Severity 2**, and it would drop to 1 under the paragraph-interruption rule in Finding 1.
- **Escape:** `1\{.} `.

*Revised 2026-08-08.* `1)` was an ordered marker and is not any more. It goes for symmetry with R8: one bullet marker, one ordered marker, list syntax stated in two lines. *(It freed nothing at the time, because `)` still closed a link destination under the old R24. Decision 20 freed `)` outright, so the symmetry argument was paid back with interest.)*

**Alphabetic and Roman markers (`a.`, `A.`, `i.`) were proposed and set aside**, and the collision is why. `A. Smith argues that…` and `J. R. R. Tolkien wrote…` are initials and author names, ordinary in citation prose, so letters would put a **severity 3** collision at the start of a line — a form that occurs naturally in writing, promoted to structure. They also carry an `i.`-is-letter-or-Roman ambiguity, no rule for what follows `z.`, and, decisively, a numbering *style* written into the source, which is what decision 9 removed classes for.

### R10 — Content column and continuation

- **Form:** an item's content column is where its content begins. Every continuation line is indented to exactly that column.
- **Range:** all list items.
- **Order:** applies while a list container is open.
- **Decidable from:** this line and the open containers.
- **Reserves:** leading spaces, as item structure.
- **Collision:** none in prose. The collision is with *habit* — Markdown's lazy continuation is removed, so an under-indented second line leaves the item instead of joining it. **Severity 2** against fluent Markdown authors, 0 against prose.
- **Escape:** not applicable; indent correctly.

### R11 — Verbatim fence

- **Form:** open with a run of three or more backticks at the start of the line's remaining content, optionally followed by a decorator list (R20). Close with a run of **exactly that length**, alone on a line. A **run** is maximal — the characters on either side of it are not backticks — and that is what every backtick count in this list means.
- **Range:** content is verbatim — no escape, no inline rule, no nested decorator. An unclosed fence closes at the end of its container.
- **Order:** 5. Wins over everything except the container prefixes.
- **Decidable from:** this line and the open containers.
- **Reserves:** a run of three or more backticks in first position.
- **Collision:** essentially none — three consecutive backticks do not occur in prose. **Severity 0.**
- **Escape:** `` \{`} `` on the first backtick.

**The run length is the only escape inside verbatim**, because R17 does not apply there. So the count is not decoration: it is what lets content hold the delimiter, at both sizes, under one rule — *choose a length that does not occur as a run in the content*. The minimum of three is unrelated to that and answers a different question: a one-backtick opener would make any paragraph starting with a verbatim span into a fence, which is prose-safety, not escaping.

*Note the family resemblance recorded at R30.* Lengthening a delimiter to hold that delimiter is now the language's only escape mechanism, shared by fences, verbatim spans, and brace groups alike.

*Changed 2026-08-08:* closing tightened from "at least as many" to exactly as many, making the counting rule the same here and in R19. New cost, and it belongs to the validator: a fence closed with too long a run is not closed and swallows the rest of its container.

*Undecided:* whether `~~~` is also a fence. See Finding 4.

### R12 — ATX heading

- **Form:** a run of one to six `#` at the start of the line's remaining content, then exactly one space, then non-empty text.
- **Range:** levels 1 to 6. No closing run, no indentation, no empty heading, no underlined form.
- **Order:** 6.
- **Decidable from:** this line and the open containers.
- **Reserves:** `#` in first position, in a run of one to six, followed by a space.
- **Collision:** a line opening with a number sign meaning "number" and a space — `# 5 is my favourite`. `#hashtag`, `#5`, `#include` and `#######` are all text, because the required space is missing or the run is too long. **Severity 2.**
- **Escape:** `\{#} 5 is my favourite`.

The required space is doing almost all the work here, and it is the cleanest example in the language of prose-safety falling out of the form rather than being carved out by an exception. Note that it also separates the heading from R23's anchor without a rule saying so: a heading needs a space after the run, an anchor needs a brace, and no line satisfies both.

### R13 — Thematic break — **REMOVED 2026-08-08 (decision 19)**

**This rule is struck, not deleted, and R13 is never reused.** A line of three or more `-`, `_`, or `*` reserves nothing; such a line is ordinary paragraph text under R16, or a bullet item under R8 where it takes that form (`- - -`).

*Struck in words rather than in marks:* strikethrough was not in the language when this rule was struck — decision 22 added `~{…}` two days later — so a struck rule says so in its heading.

**What it read before removal**, kept so that a note citing R13 still resolves:

- **Form:** three or more `-`, `_`, or `*`, alone on a line, spaces permitted between them.
- **Order:** 7, but ahead of R8 when a line satisfied both (`- - -`).
- **Reserves:** a line consisting only of those characters.
- **Collision:** an ASCII horizontal rule written by hand. **Severity 1.**

**What the removal returned.** `*` and `_` each got their line-start column back, and **R8's order line lost its exception** — the "ahead of R8 when a line satisfies both" clause was the only precedence contest in Layer 1, and an ordering exception is an "unless" clause wearing a different hat.

### R14 — Table

- **Form:** any line beginning with `|` opens a table. Cells are separated by `|`; a line beginning with `|` and carrying content holds inline cells, and a `|` alone on a line opens a **block cell** whose content is every following line up to the next line beginning with `|`. Rows form by counting cells to the column count.
- **Range:** the column count comes from the first row, rule row or not; a single-column table needs none. **A rule-row cell is three or more `-`, optionally carrying `:` at one end or both** — leading colon left, trailing colon right, both centred, bare run no preference — and a rule row is recognized only as the table's first or second row. Its position says whether there is a header: first line, no header; second row, the row above was the header; absent, neither. **The table ends at a blank line immediately following a line beginning with `|`**, or at the end of its container. The closing mark, a `|` alone on a line before a blank line, opens no cell. Leading and trailing pipes are mandatory on structure lines.
- **Order:** 7.
- **Decidable from:** this line and the open containers.
- **Reserves:** `|` in first position, and `|` as a cell separator inside a table block.
- **Collision:** a line of prose or code beginning with a pipe — BNF alternation, a shell pipeline, `|x| < 5` — is a one-cell table rather than a paragraph. **Severity 2.** All three cases are technical rather than everyday, all three belong in a verbatim span anyway, and the failure is visible rather than silent: the text survives and acquires a border. It would drop back to 1 under the paragraph-interruption rule in Finding 1.
- **Escape:** `\{|}`.

*Rewritten 2026-08-08, and this rule is where the design paid for itself twice.* **Finding 2 is closed** because a table now opens on its own first line like every other block, so nothing in Layer 1 looks ahead. And the **leading pipe becomes mandatory by construction**, closing a prose-safety hole GFM leaves open, where an optional leading pipe makes `a | b` in running prose a valid header row.

*Note on the rule row.* GFM accepts a **single** hyphen in a rule-row cell, and gets away with it because its delimiter row can only ever be a table's second line. Once a rule row may also come first, `| - | - |` opening a headerless table parses as structure and **those cells vanish** — a **severity 4**, the only class this language treats as disqualifying. Two changes remove it: a three-hyphen minimum, matching R11, and recognizing a rule row only as the table's first or second row. The residual exposure is a cell whose genuine content is a dash run, in one of those two positions, which needs `\{---}`. **Severity 1.**

*Note on the terminator.* Keying the end of a table on a line **ending** with `|` fails on a block cell whose paragraph ends with a pipe: `Write the alternation as a |` would close the table and drop the rest of the cell into the document. Keying on the **first** character adds no hazard the cell rule had not already created.

### R15 — Link reference definition — **REMOVED 2026-08-09 (decision 20)**

**This rule is struck, not deleted, and R15 is never reused.** A line beginning with `[` reserves nothing. Reference links went with it (R25), so there is nothing for a definition to define.

**What it read before removal:**

- **Form:** `[label]: destination` optionally followed by a title, alone in its own block.
- **Order:** 9.
- **Reserves:** a line beginning with `[` when the bracket closes and is followed by `:`.
- **Collision:** a line of prose opening with a bracketed citation followed by a colon — `[Smith]: the argument runs…`. **Severity 1.**

**What the removal returns.** `[` leaves the line-start column, which is what left that column holding exactly the six block markers *(seven since decision 23 added the aside)*. The capability is unchanged in substance: a link's target is written where the link is, and a target used repeatedly is a repetition rather than a definition. See R25 for why the shortcut form was the real cost.

### R16 — Paragraph

- **Form:** anything else. Inline content (Layer 2) is parsed inside it.
- **Range:** until a blank line or the end of the container.
- **Order:** last.
- **Decidable from:** nothing — the fallback.
- **Reserves:** nothing. **Severity 0.**
- **Escape:** not applicable.

### R31 — Task item

- **Form:** inside a bullet item (R8), content beginning with a **state box** — `[`, then one space or one `x`, then `]`, then one space. `[ ]` is unchecked; `[x]` is checked.
- **Range:** bullet items only; lowercase `x` only; the box is exactly three characters. The item's content column (R10) sits after the box's trailing space. Anything failing the form — `[X]`, `[x ]`, a box on an ordered item — is ordinary item text.
- **Order:** after R8 consumes its marker, before the item's content is parsed. *(Numbered past Layer 2 because rule numbers are serial and permanent; it belongs here.)*
- **Decidable from:** the four code points after the marker's space.
- **Reserves:** `[` at the start of a bullet item's content, in exactly the forms `[ ] ` and `[x] `.
- **Collision:** an item genuinely beginning with a bracketed box — a quoted checklist, or `- [x] marks the spot`. **Severity 1.**
- **Escape:** `\{[x]}`, or `\{[}` on the bracket alone.

*Added 2026-08-10 — decision 21.* A refinement of R8 rather than a new block marker: the line-start column gains nothing here, and the box sits inside an item the marker already opened. The spelling is Markdown's because the reader's convention governs (decision 18's argument) — `[ ]`/`[x]` is the forty-year plain-text checkbox. `-[x]` and `-{x}` were both set aside: the first for the failure asymmetry (every GFM-trained hand types the space form, so the space form must be the construct), the second because sigil-then-brace is Layer 2's rule and the block layer holds no brace anywhere. GFM's looseness is removed — `[X]` is text, with a validator warning and a formatter fix. Task lists are also the first capability to *fail* §2's routing test, which is why the grammar carries them; the argument is in the decision.

### R33 — Aside marker

- **Form:** `<` at the start of the line's remaining content, then one space, then content — or `<` alone on the line, the aside's blank line.
- **Range:** a container, as R7: nests without limit and holds any blocks.
- **Order:** 2, as a container prefix alongside R7; the two share a shape, and no input satisfies both. *(Numbered past Layer 2 because rule numbers are serial and permanent; it belongs here.)*
- **Decidable from:** this line.
- **Reserves:** `<` in first position when followed by one space, or alone on the line.
- **Collision:** a line of prose opening with a spaced comparison — `< 5 ms is the target`. **Severity 1.** `<div>`, `<!DOCTYPE`, `<T>`, and `<1%` are all text, the required space missing.
- **Escape:** `\{<} 5 ms`.

*Added 2026-08-10 — decision 23.* **The converse of the block quote, and the mirror is the meaning:** `>` marks content that presses harder than the prose around it, `<` marks content that presses less — an aside, liftable with little impact — and both mark the left of every line they own. Rendering is the renderer's choice: a sidebar, a margin note, smaller type, or a disclosure fold on an interactive target; folding is one rendering of the semantic, never the construct, which is what keeps the rule meaningful in print. **The mirror is exact: R7 requires its space too** *(amended the same day — the asymmetry as first recorded lasted hours)*, so the two containers are one rule with two characters. The requirement is what makes a pasted `<div>` or `<!DOCTYPE` at line start ordinary text by construction rather than by luck — and `>5ms` and `>=` likewise, on the other side. No title line — a folding renderer may use the first line by convention — and no type vocabulary: decision 9 removed classes, and "Note:" is text.

## Layer 2 — Inlines

Applied within any block that takes inline content. Verbatim content (R11, R19) is never reached by any rule in this layer.

**One rule governs this layer: a sigil means nothing unless the very next character is `{`** — extended by `[` for the link and image forms that carry their own text. R30 states the shared machinery; R17, R20, R21, R23, R24, R27, R29, and R32 are its instances, and they differ only in which sigil opens them and what the brace holds.

**Order barely matters here, and that is the point.** A left-to-right scan reaches a sigil before it reaches anything else, and no two constructs share an opener, so the numbering below records the parser's convenience rather than a precedence contest. Every rule in this layer is decidable from its sigil run and the code point after it — the verbatim span alone also needs the search for its matching run (R19), which is the price of uninterpreted content.

### R17 — Escape range

- **Form:** `\{`, literal content, `}`. The content is text: no rule in this layer applies inside it.
- **Range:** the content is literal but brace-balanced (R30). `\` followed by anything other than `{` is an ordinary backslash — except before a line ending, which is R18.
- **Order:** 1. Before everything, so an escape range can defeat any rule below.
- **Decidable from:** two code points.
- **Reserves:** `\` only when immediately followed by `{`.
- **Collision:** `\{` in TeX, where it is a literal brace, quoted in prose outside a fence. **Severity 1.**
- **Escape:** lengthen the run — `\{{…}}` — or use a verbatim span.

*Rewritten 2026-08-09 — decision 20, and this is the rule the decision was worth making for.* The previous form was `\` followed by *any* single code point, yielding that code point with no exception list. It read perfectly and it was the only rule in the language that made ordinary writing worse than CommonMark does: a backslash before a letter survives under CommonMark, so `C:\Users\frederic`, `\n`, and `\d+` are safe there and were mangled here, silently. That was **severity 3** on this scale and arguably 4, since nothing on the page showed it had happened. **Finding 3 is closed** — it had proposed a validator warning, and the cause was removed instead.

Two consequences worth stating, because neither is obvious from the form:

**The escape is now visible.** `a \{|} b` says *I meant this pipe*, where a lone `\|` in a paragraph reads as a typo. The range delimits what is protected, so a search-and-replace that empties one leaves `\{}` behind rather than a stray backslash.

**The need for it collapsed.** Under the doublet rule almost nothing in ordinary writing needs escaping at all, which is why a heavier escape costs less than the lighter one did.

### R18 — Hard line break

- **Form:** `\` immediately before a line ending.
- **Range:** within a paragraph. It is the only hard break; trailing spaces are not one.
- **Order:** 2.
- **Decidable from:** two code points.
- **Reserves:** `\` at end of line.
- **Collision:** a line of prose ending in a backslash — a shell continuation quoted inline. **Severity 1**, and it is the one collision in the language whose failure mode is *correct*: the author wanted a line break there anyway.
- **Escape:** `C:\{\}` — an escape range holding a backslash.

**Unchanged by decision 20, and it is not the exception to left-marking it appears to be.** `\` before a line ending is the sigil operating on the line ending, sitting immediately to its left, which is R17's shape exactly. So `\` takes two operands, a brace range and a line ending, and the second is structurally forced rather than chosen: a line ending is the one character that cannot appear inside a brace range, because a brace range is inline.

*Alternatives considered and set aside in the same session.* `\\`, matching TeX, is exactly as invisible as `\` at end of line — so it buys nothing on intentionality — and it puts UNC paths (`\\server\share`) and escaped backslashes in quoted code into the blast radius. `\n` would reintroduce the exception list R17 was rewritten to remove. `\{}` is a no-op and correctly so: an escape range with no content renders nothing.

### R19 — Verbatim span

- **Form:** a run of one or more backticks, content, then a run of the same length. Runs are maximal, as in R11, and the length may go down as well as up — a one-backtick span may hold a doubled run.
- **Range:** content is verbatim; no rule below applies inside it.
- **Order:** 3. Wins over every construct below, which is what makes it the universal safe harbour.
- **Decidable from:** the run length and the search for its match within the block.
- **Reserves:** the backtick, everywhere.
- **Collision:** a backtick in prose — a grave accent typed deliberately, or an apostrophe mistyped as `` ` `` or `´`. **Severity 1**, offset by a real accessibility cost on non-US keyboards that is answered with tooling rather than syntax.
- **Escape:** `` \{`} ``.

**This is the one construct that is delimited on both sides rather than marked on the left, and the reason is its own semantics.** Verbatim content is not interpreted, so its closing delimiter cannot be found by balancing — closing the span would mean reading the content. Run-length matching is the only escape that works without looking inside. That is why verbatim keeps the paired form and nothing else does, and why R30 could not absorb it.

*Undecided:* whether CommonMark's strip-one-leading-and-trailing-space rule is inherited. It has an exception clause, so invariant 2 argues against.

### R20 — Decorator list

- **Form:** a space-separated token list, in exactly two shapes and at most one of each: a bare word (the format label) and `#identifier` (an anchor). Written after a fence's opening backticks, or in braces immediately after a verbatim span's closing run.
- **Range:** tokens exclude whitespace, control characters, `{`, `}`, and the backtick. Nothing else is excluded. The first character selects the shape.
- **Order:** 4, and **only** in those two positions.
- **Decidable from:** the preceding backtick run.
- **Reserves:** `{` immediately after a closing backtick run.
- **Collision:** none in prose, because the position requires a preceding verbatim span. **Severity 0.**
- **Escape:** not needed; put a space before the brace.

No word is ever reserved, so no decorator token can affect the parse. That is what keeps this rule's collision surface at zero permanently rather than at zero for now.

*Changed 2026-08-09.* `(` and `)` leave the exclusion list. They were excluded because they closed a link destination under the old R24; decision 20 moved link targets into braces, so parentheses are ordinary characters in a decorator token as they are everywhere else.

*Changed 2026-08-08:* the backtick joins the exclusion list. On a fence the list runs to the end of the line, so a backtick in a token would make ```` ```rust``` ```` an opener whose label ends in a run — a line that reads as a closed empty fence and is not one. Inline it would put an unpaired run into R19's scan.

**The braces delimit the list; they are not part of any token.** A fence needs no delimiter, because the rest of the line is the list, so the fence form carries none — ```` ```rust #id ````, never ```` ```rust {#id} ````.

**This is the one place in the language where the mark follows what it marks.** A decorator annotates the span before it, so the arrow points backwards even though the brace group is still opened by the sigil to its left. That is annotation rather than marking, and the asymmetry is deliberate: a fence already delimits, so there is nothing for a brace to scope.

*Undecided:* what a line whose decorator list is malformed becomes — paragraph text under the uniform fallback, or a fence carrying a validator error. Three cases reach it: a token holding a backtick, two bare words, two anchors.

### R21 — Emphasis, strong, and both

- **Form:** `*{`, inline content, `}` emphasizes. `**{` is strong; `***{` is both.
- **Range:** runs of one, two, and three `*` are defined. A run of four or more is not a construct, so the run and the brace are both text — the same shape as R12's six-`#` ceiling. Nesting works by R30.
- **Order:** 5.
- **Decidable from:** the maximal `*` run and the code point after it.
- **Reserves:** `*` only when a run of one to three is immediately followed by `{`.
- **Collision:** shell brace expansion after a glob — `ls *{png,jpg}` — quoted in prose outside a fence. **Severity 1.**
- **Escape:** `\{*}{png,jpg}`, or a verbatim span, which is where a shell snippet belongs.

*Rewritten 2026-08-09 — decision 20.* The previous form was `*text*` and `**text**`, with flanking conditions on both delimiters. **This removed the last heuristic in the language.** Flanking decided whether `*` was a mark or a character by looking at what surrounded it, which is prose-safety by rule rather than by construction, and invariant 1 promises the second. It was also the only place where a reader could not tell from the mark itself what the mark was doing, and `2*3*4` emphasized the middle regardless.

Three things came with it, none of them the point but all of them worth recording:

**CommonMark's hardest algorithm is gone.** The delimiter stack, left- and right-flanking classification, and the rule of 3 exist to resolve emphasis. `*{a **{b} c}` is unambiguous by balance alone.

**`***{…}` falls out of the run rule** rather than needing the nesting corner CommonMark cannot predict.

**The typing cost is one character, on one of three forms.** `*em*` to `*{em}` is `+1`; `**em**` to `**{em}` is free; `***em***` to `***{em}` is `−1`.

### R22 — Bracketed emphasis — **REMOVED 2026-08-09 (decision 20)**

**This rule is struck, not deleted, and R22 is never reused.** It was `{*text*}`, the intra-word form, and R21 now has the same shape with the brace on the other side. Nothing is lost: the construct existed because paired `*` cannot reach inside a word, and R21's braced form reaches everywhere.

R22 is the rule that should have been read as a signal rather than a special case. It and R23 already put a brace and a sigil together, and R29 then did the same in the other order; decision 20 is the observation that all three were one rule written three times.

### R23 — Anchor

- **Form:** `#{`, an identifier, `}`. Valid anywhere inline content is valid. Marks a point, not a construct.
- **Range:** identifier character set as R20. Whitespace on either side is optional and insignificant.
- **Order:** 6.
- **Decidable from:** two code points.
- **Reserves:** `#` only when immediately followed by `{`.
- **Collision:** Ruby and Elixir string interpolation quoted in prose outside a verbatim span — `"Hello #{name}"`. **Severity 1**, and the sharpest of the new doublet collisions, since it is an idiom rather than a coincidence.
- **Escape:** `\{#}{name}`, or a verbatim span, which is where interpolated code belongs.

*Rewritten 2026-08-09 — decision 20.* The previous form was `{#identifier}`, brace-first. Turning it around cost nothing in the construct and bought the general rule, and it moved the collision: Svelte and Handlebars `{#if}` and `{#each}` are now ordinary text, and Ruby interpolation takes their place. That is close to a wash on the same audience, and it is recorded rather than claimed as a win.

**References still need no new syntax.** `@[text]{#five-minute}`, `@[text]{definition.md#five-minute}`, and `@[text]{https://example.org/c#five-minute}` are one construct — a link whose target carries a fragment.

### R24 — Link

- **Form:** `@{`, a target, `}` — or `@[`, inline text, `]{`, a target, `}` when the link carries its own text.
- **Range:** the target may be a URL, a path, or a fragment; it is literal and never parsed as inline content. Bare `@{target}` renders the target as its own text.
- **Order:** 7.
- **Decidable from:** two code points.
- **Reserves:** `@` only when immediately followed by `{` or `[`.
- **Collision:** `@{` in prose is essentially unattested; `@[` likewise. **Severity 1.**
- **Escape:** `\{@}{handle}`.

*Rewritten 2026-08-09 — decision 20, and this rule carries the largest single delta from Markdown in the language.* The previous form was `[text](destination)`, and its removal is what buys three separate things:

**The last inline lookahead closes.** Under `[text](url)` a parser must scan to `]` and then check for `(` before it knows whether the bracket opened anything. `@[` decides on the second code point, which makes Layer 2 prefix-decidable in the same sense R12 makes Layer 1.

**Bare `[` becomes prose-safe by construction.** `[sic]`, `[1]`, and `[citation needed]` rendered literally under CommonMark *because no `(` followed* — a lookahead-based rescue, which is exactly what invariant 1 says the language does not do.

**Parentheses come back.** `(` and `)` return to the free column, and CommonMark's balanced-paren rules for destinations go with them.

**The two delimited parts are not an inconsistency, they are the reading order.** `@[text]{target}` puts what you see in brackets and what it means in braces — the same order as `` `x=y`{math} ``, where the span is what you see and the label is what it means. R24 and R27 are the only rules where a sigil's next character may be `[` rather than `{` — the stated extension on the one-line rule.

### R25 — Reference link — **REMOVED 2026-08-09 (decision 20)**

**This rule is struck, not deleted, and R25 is never reused.** It was `[text][label]`, `[text][]`, and the shortcut `[text]`, resolved against R15.

**What the removal returns, and it is the largest prose-safety gain in the change after R17.** **Finding 6 is closed.** The shortcut form was the only construct in the language with action at a distance: `[sic]`, `[1]`, `[TODO]`, and `[citation needed]` are ordinary prose, and any of them became a link if a definition with a matching label existed anywhere in the same document — including one added months later by someone who never saw the paragraph they changed. **Severity 2, rising with document length**, decidable only against a document-wide table, and silent when it fired. Nothing else in the language behaved that way, and nothing does now.

The cost is a real one and it is not hidden: a URL used twenty times is written twenty times. That is what the canonical formatter and the editor are for, and it is the same answer tables of contents got — the tooling writes into the source.

### R26 — Autolink — **REMOVED 2026-08-09 (decision 20)**

**This rule is struck, not deleted, and R26 is never reused.** It was `<scheme:rest>` and bare email addresses in angle brackets.

`@{https://example.org}` covers the case exactly, and covers it better: a bare `@{target}` renders the target as its own text, which is what an autolink did, without a second syntax for it. **`<` returns to the free column** as a result, so `a < b`, `Vec<T>`, and `<div>` quoted in prose are text by construction rather than because no scheme followed.

### R27 — Image

- **Form:** `!{`, a source, `}` — or `![`, alt text, `]{`, a source, `}`.
- **Range:** alt text optional; empty alt well-formed, missing alt is lint.
- **Order:** 8.
- **Decidable from:** two code points.
- **Reserves:** `!` only when immediately followed by `{` or `[`.
- **Collision:** `!{` or `![` in prose. **Severity 1.**
- **Escape:** `\{!}{note}`.

*Rewritten 2026-08-09 — decision 20.* The previous form was `![alt](src)`, unchanged from CommonMark; it is now the same delta as R24 and for the same reasons. **The mnemonic survives intact and is the reason the parallel matters:** `@[…]` links to the target, `![…]` shows it instead. The `!` is a presentation switch on the same construct, not a different one, and the two rules now differ by exactly one character rather than by a bracket-and-paren dance.

### R28 — Text

- **Form:** everything else.
- **Order:** last.
- **Collision:** none. **Severity 0.**

### R29 — Superscript and subscript

- **Form:** `^{`, inline content, `}` raises the content; `_{`, inline content, `}` lowers it.
- **Range:** the content is ordinary inline content, so nesting works and emphasis inside it works. There is no braceless form and no length limit. The construct positions content and interprets nothing else — it is not mathematics, and no implementation routes it through a math engine.
- **Order:** 9.
- **Decidable from:** two code points.
- **Reserves:** `^` and `_`, each only when immediately followed by `{`.
- **Collision:** raw TeX pasted into prose outside a fence — `\int_{0}^{\infty}` — where `_{0}` and `^{\infty}` both fire. **Severity 1**, and R17 no longer compounds it: under the rewritten escape rule the backslashes in that string survive, so only the two braced runs are affected. Mustache and Handlebars `{{^inverted}}` no longer collide at all, the sigil now needing a brace *after* it.
- **Escape:** `\{^}` and `\{_}`.

`2^32`, `[^0-9]`, `ISO_8601`, `file_2.txt`, `UTF_8`, and `snake_case` are all still text, needing no escape, because none of them puts a brace after the sigil.

An unclosed `^{` is text, under the uniform fallback. `^{}` is well-formed, renders nothing, and is a lint.

**What this rule does not do is the half that keeps it cheap.** It applies a baseline offset and a size reduction, and no other typography: no italics, no math spacing, no symbol interpretation. That is not a restriction bolted on, it is the definition, and it is what stops the construct from being a second way to write mathematics — `E=mc^{2}` and `` `E=mc^2`{math} `` render *differently*, the first as roman body text with a raised 2 and the second as typeset math with italic variables.

The rule self-sorts as a result. A variable set upright looks wrong to the person writing it, so `x^{n}` pushes its author toward `math`, where variables belong; digits, units, ordinals and chemical formulae look right upright, so they stay in prose. Chemistry is the case where upright is not an approximation but the correct typesetting — `$H_2O$` renders an italic H and is wrong, which is why mhchem exists.

*Added 2026-08-08 — decision 18. Generalized 2026-08-09 — decision 20.* This rule was written for two characters and turned out to be the shape of the whole layer. It is unchanged in form; what changed around it is that it stopped being a special case.

### R30 — The brace group

- **Form:** a sigil, then a run of one or more `{`, then content, then a run of **exactly the opening length** of `}`.
- **Range:** the shared machinery of R17, R20, R21, R23, R24, R27, R29, and R32 — R20's "sigil" being the closing backtick run it follows. Runs are maximal, as in R11 and R19.
- **Order:** not applicable — a definition, not a match. Every rule in this layer is an instance of it.
- **Decidable from:** the sigil and the code point after it.
- **Reserves:** `{` and `}` only immediately after a sigil. A brace with no sigil before it is text, everywhere, always.
- **Collision:** none of its own; each instance carries its own. **Severity 0.**
- **Escape:** lengthen the run.

**Two flavours of content, and this is the only distinction that reaches the parser.** *Inline content* — R21, R29, R32 — is parsed recursively, so a nested construct is consumed whole and its braces never reach the outer close. *Literal content* — R17, R23, R24, R27, R20 — is not parsed, so the close is found by counting brace depth. Both are linear with no backtracking, and both honour the same escape.

**Single braces work whenever the content is balanced.** `\{a {b} c}` closes at the final `}` with no lengthening at all. Lengthening is the escape hatch for the case that actually fails: `*{the } character}` breaks, and `*{{the } character}}` does not, because a lone `}` is a run of one and cannot close a run of two.

**This makes the language's escape story one sentence: when a delimiter appears in your content, lengthen the delimiter.** Fences (R11), verbatim spans (R19), and brace groups, identically. There is no second mechanism, and R17's escape range is not one — it is a construct that happens to hold literal text, not a way of escaping a character.

*Phase 1 owes this rule a worked example* of the corner where the two flavours meet: an inline-content group containing a literal-content group whose own content holds an unbalanced brace. The behaviour follows from the two paragraphs above, and it should be written down rather than derived.

### R32 — Strikethrough

- **Form:** `~{`, inline content, `}`.
- **Range:** a run of exactly one `~` before the brace. A run of two or more is not a construct, so the run and any brace after it are text — the same fallback shape as R21's four-asterisk ceiling, which is what makes GFM's `~~struck~~` literal text here, visibly. Content is ordinary inline content; nesting works by R30.
- **Order:** 10.
- **Decidable from:** the maximal `~` run and the code point after it.
- **Reserves:** `~` only when alone and immediately followed by `{`.
- **Collision:** shell tilde-and-brace expansion quoted in prose outside a fence — `~{alice,bob}/bin`. **Severity 1.**
- **Escape:** `\{~}{alice,bob}`, or a verbatim span, which is where a shell snippet belongs.

*Added 2026-08-10 — decision 22.* The single-meaning sigils are run-1 throughout, and strikethrough has no ladder, so one tilde is the construct and `~~{…}` is deliberately nothing — a dead minimal run would be the only one in the language. Typographic strike only, decision 18's boundary reapplied: the construct decorates content and interprets nothing. `~` in prose — `~10`, `~/bin` — never precedes a brace, which is what kept the price at severity 1. Closes the strikethrough half of Finding 5.

## Findings

Things this list has surfaced that the decision record does not settle. Each is a candidate for the record; none is decided here. Closed findings keep their number and a pointer.

### Finding 1 — "No block interrupts a paragraph" would cut the collision surface substantially, and is not on the record

`.claude/landscape.md` lists "block elements can't interrupt paragraphs" among djot's design principles that we share. It appears nowhere in `.claude/decisions.md`, and `definition.md` does not state it.

If adopted, a line inside an open paragraph is text no matter what it looks like, and R8, R9, R12, and R14 all stop firing mid-paragraph. That drops R9 from severity 2 to 1, softens R12, and removes the whole class of "my paragraph turned into a list because it started with a year".

The cost is a real surprise in the other direction: a list written immediately under a line of prose, with no blank line between, stops being a list.

**Worth deciding explicitly, either way**, and it is now the largest remaining item on this list — decision 20 closed the two findings that outranked it.

### Finding 2 — CLOSED 2026-08-08 — pipe tables needed one line of lookahead

**Resolved by design change, not by amendment.** Any line beginning with `|` now opens a table, so a table is decidable from its own first line like every other block.

Kept because the shape of the resolution is worth reusing: **when a finding here points at an invariant, try moving the construct before moving the invariant.** Decision 20 is the second instance, and a much larger one.

### Finding 3 — CLOSED 2026-08-09 — decision 3 was the only decision that *increased* the collision surface

**Resolved by removing the cause.** This finding recorded that uniform escaping made `C:\Users`, `\n`, and `\d+` unsafe in running prose where CommonMark leaves them safe, and proposed a validator warning as mitigation. Decision 20 rewrote R17 as the escape range instead, so the backslash is inert unless a brace follows it and the mitigation is unnecessary.

Kept because it is the clearest instance of a pattern worth watching for: **a rule can read perfectly and still be the worst rule in the language.** Nobody noticed this one for two days because "no exception list" is such a good sentence.

### Finding 4 — Tilde fences are undecided

CommonMark admits `~~~` as an alternative fence. The decision record never mentions it, `deltas.md` has no row for it, and every statement of decision 9's decorator grammar is written in terms of backticks.

Removing them is what the one-way-to-do-it discipline argues for, and under decision 20's clean break it is no longer even a delta worth counting. The knock-on is restated after decision 22 spent `~` on strikethrough: the "keeps `~` meaningless anywhere" support is gone, so the as-drafted answer rests on the discipline argument alone — and admitting tilde fences would still spend `~`'s line-start column, which strikethrough does not touch. *`definition.md` Appendix D.5 now states the as-drafted answer — not inherited — pending confirmation; confirming it closes this finding.*

### Finding 5 — HALF-CLOSED 2026-08-10 — strikethrough is decision 22; entity references remain absent by accident

- **Strikethrough — CLOSED.** Adopted as `~{…}`, single tilde, R32. The cost this finding predicted — `~` losing its free column — was paid knowingly, at severity 1 and inline only.

- **HTML entity references (`&amp;`, `&#169;`)** should almost certainly go: an entity reference renders a character the source does not contain, which is decision 15's front half through a very small door, and resolving one requires an HTML entity table, a dependency on a serialization the language claims independence from. Under the clean break the compatibility argument for keeping them is gone entirely.

The entity half still needs its line in the record, and it is the substantive one. *Held as `definition.md` Appendix D.14, whose strikethrough half decision 22 closed.*

### Finding 6 — CLOSED 2026-08-09 — shortcut reference links were the only construct with action at a distance

**Resolved by removing the construct.** R25 is struck and R15 with it. See R25 for what that returns.

### Finding 7 — the language has no inline construct that spans a line ending

Every Layer 2 rule opens and closes within one block, and R30's brace groups are no exception — an unclosed group is text under the uniform fallback. Nothing says whether a brace group may contain a line ending, which matters for a long link target or a wrapped emphasis in a hard-wrapped document.

Raised by the R30 rewrite. It has a natural answer — a paragraph is one logical line, so a group closes anywhere within it — but nothing states it, and an implementer would have to guess. *Held as `definition.md` Appendix D.12.*

## Maintenance

Add a rule when a decision creates one, in the same session that records the decision. **Update the reserved-position table in the same edit.** It is the first thing in the file because it is the only part consulted while writing rather than while designing, so it is the part most read and the part where staleness does real damage — a wrong row tells someone a character is safe to type when it is not.

**A row is placed by its severity**, the table running from the biggest cost to plain text down to none. So a character whose severity rises moves *up* the table in the same edit that raises it, and one that is bought back falls to the bottom. The sort is the ledger: a decision's price and its dividend are both visible as movement, before anyone reads the rule that caused it.

**Update the doublet table too** when a decision adds or removes an inline construct. It is the reference card's inline half, and it is the only place the whole of Layer 2 appears at once.

When a finding here is settled, replace its body with a one-line pointer to the amendment in `.claude/decisions.md`, or — better, and twice now — record that the construct moved instead.
