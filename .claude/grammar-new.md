# Grammar sketch — MOCK, brace doublets

*Mock only. Not the running rule list — that is `.claude/grammar.md`, which is unchanged. This file exists to show what the reserved-position table looks like under the proposal in `notes/brace-doublets.md`, before anything is implemented. Delete it or promote it; do not let both files be authoritative.*

## The doublet table

Every operative inline construct, and the two-character sequence that opens it. **A sigil means nothing unless the very next character is `{`.**

| Opener | Construct | Written | Content |
|---|---|---|---|
| `*{` | emphasis | `*{em}` | inline |
| `**{` | strong | `**{strong}` | inline |
| `***{` | both | `***{both}` | inline |
| `^{` | superscript | `10^{23}` | inline |
| `_{` | subscript | `H_{2}O` | inline |
| `#{` | anchor | `#{tessier-2026}` | literal |
| `@{` | link | `@{https://example.org}` | literal |
| `@[` | link with text | `@[Tessier et al.]{#tessier-2026}` | literal target |
| `!{` | image | `!{assets/kangaroo.png}` | literal |
| `![` | image with alt | `![a kangaroo in Cairns]{assets/kangaroo.png}` | literal target |
| `\{` | escape range | `a \{|} b` | literal |
| `` `{ `` | format label | `` `x=y`{math} `` | literal |

Ten sigils, one shape. The **Content** column is the only distinction that reaches the grammar: inline content is parsed recursively, literal content counts brace depth. Neither changes what an author types.

## The reserved-position table

Every ASCII punctuation character, and what it costs plain text. This is the one-page statement invariant 1 promises, and it is first in the file because it is the part you consult while typing — what is safe to write, at a glance, without reading a rule.

**Rows run from the biggest cost to plain text down to none**, so the top of the table is what to watch when writing and the bottom is what is always safe.

| Character | At the start of a line | Inside a line | Worst severity |
|---|---|---|---|
| `-` | bullet marker, then a space | free | 3 |
| `#` | heading, run of 1–6 then a space | anchor, only before `{` | 2 |
| digits | ordered marker, then `.` and a space | free | 2 |
| `\|` | opens a table | cell separator inside a table only | 2 |
| `` ` `` | fence, run of 3+ | verbatim span, any run; format label before `{` | 1 |
| `>` | block quote | free | 1 |
| `\` | free | escape range, only before `{`; hard break at end of line | 1 |
| `*` | free | emphasis, only before `{`, runs of 1–3 | 1 |
| `_` | free | subscript, only before `{` | 1 |
| `^` | free | superscript, only before `{` | 1 |
| `@` | free | link, only before `{` or `[` | 1 |
| `!` | free | image, only before `{` or `[` | 1 |
| `[` `]` | free | link or image text, only immediately after `@` or `!` | 1 |
| `{` `}` | free | scope, only immediately after a sigil | 1 |
| `(` `)` | **free** | **free** | **0** |
| `<` | **free** | **free** | **0** |
| `+` | **free** | **free** | **0** |
| `$` | **free** | **free** | **0** |
| `~` | **free** | **free** | **0** |
| `%` `:` `;` `?` `/` `=` `&` `"` `'` `,` `.` | **free** | **free** | **0** |

**Seventeen** of the thirty-two ASCII punctuation characters have no meaning anywhere in the language — the eleven in the last row, plus `(`, `)`, `<`, `+`, `$`, and `~`.

**`-` is the only severity-3 character left**, and it is the only one that reaches band 3 in either position.

**The line-start column holds exactly the six block markers** — `` ` ``, `#`, `-`, digits, `>`, `|` — and nothing else. Class 1 of the language, stated as a table.

### The ledger against `.claude/grammar.md`

One character spent, six earned, two bands emptied.

| Character | Now | Mock | What moved |
|---|---|---|---|
| `@` | **0** | 1 | **spent** — link, before `{` or `[` |
| `\` | 3 | 1 | escape narrowed to `\{`; the last severity-3 meaning inside a line |
| `*` | 2 | 1 | flanking gone; operative only before `{` |
| `[` `]` | 2 | 1 | operative only after `@` or `!`; leaves the line-start column |
| `{` `}` | 2 | 1 | operative only after a sigil |
| `(` `)` | 1 | **0** | no construct uses parentheses |
| `<` | 1 | **0** | autolinks removed |

The `\` row is the one that matters most and it is not a consistency gain — it is a defect closing. `C:\Users\frederic` renders as `C:Usersfrederic` today, because a backslash escapes any character with no exception list. That is ordinary technical writing altered with no visible cue, which is severity 4 on the scale below, and invariant 1 does not admit band 4 at all.

The `<` row assumes reference links and autolinks go with the clean break; that question is open (`notes/brace-doublets.md`, question 3). If they stay, `<` holds at 1 and `[` keeps its line-start meaning.

### How to read the severity column

| # | Name | Meaning |
|---|------|---------|
| 0 | none | the character has no meaning in this position, guaranteed |
| 1 | negligible | requires a spelling essentially nobody produces by accident |
| 2 | occasional | shows up in technical prose from time to time |
| 3 | frequent | shows up in ordinary prose regularly |
| 4 | hazard | silently changes meaning in common writing, with no visible cue |

**The goal is zero 4s and as few 3s as the language can manage.**

### The severity-1 collisions this introduces, named

Every doublet is a new two-character sequence that ordinary writing can produce. Two are worth naming rather than smoothing over:

- **`#{`** is Ruby and Elixir string interpolation, so it appears in prose about those languages: `"Hello #{name}"`.

- **`*{`** is shell brace expansion after a glob: `ls *{png,jpg}`.

Both are answered by a verbatim span, which is where such text belongs. Neither is close to the band the removed collisions occupied, and the rest — `^{`, `_{`, `@{`, `!{`, `\{`, `` `{ `` — have no idiom behind them at all.

## Where this file stops

The rule list itself (R1 onward) is not reproduced here. Under the proposal it changes as follows, and the work is Phase 1's:

- **Rewritten:** R17 (backslash escape → escape range), R21 (emphasis), R22 (bracketed emphasis → the only form), R23 (anchor), R24 (inline link), R25, R27 (image), R29 (superscript and subscript).
- **Struck, if question 3 goes that way:** R15 (link reference definition), R26 (autolink).
- **Unchanged:** R1–R14, R16, R18 (hard break), R19, R20, R28.
- **New:** the brace run-length close, shared by every construct in the doublet table above.

R18 is worth calling out as unchanged. `\` before a line ending stays the hard break, which means the largest syntax change in the project's history ships with zero migration cost on the one delta a migrator cannot verify by eye.
