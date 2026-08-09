# Deltas from Markdown

*Crown copyright, Canada — 2026. Licensed CC BY 4.0.*

**Status: scaffold.** The delta list below is complete as to *which* decisions diverge from CommonMark, because those decisions are settled. The per-delta detail — exact before and after, edge cases, migrator behaviour — is not written yet, and **no text in this file is normative until it is.**

**Read this first.** Under decision 12, Markleft is not Markdown-compatible. Every inline construct differs: links, images, emphasis, anchors, and escapes. A Markdown document does not parse as intended here, and a Markleft document does not render as intended there. This file is therefore a **translation guide** — a map between two languages for a reader who already knows one of them — and not a promise that a document ports. It is Appendix A of `definition.md`.

**Row numbers here are not stable identifiers.** They renumber whenever a delta is inserted, which has happened repeatedly, so cite the *decision* number when referring to a divergence anywhere outside this file. Decision numbers never move; these do.

## What does not change

The shortest useful summary of the whole table: **Markleft keeps Markdown's block layer and replaces its inline layer.** A Markdown writer's fingers are right about all six block constructs and wrong about every inline one.

| Construct | Markdown | Markleft |
|---|---|---|
| heading | `## Title` | `## Title` |
| bullet | `- item` | `- item` |
| ordered item | `1. item` | `1. item` |
| block quote | `> quoted` | `> quoted` |
| fence | ```` ```rust ```` | ```` ```rust ```` |
| table | `\| a \| b \|` | `\| a \| b \|` |
| verbatim span | `` `code` `` | `` `code` `` |

Those are tightened rather than reshaped — decisions 4, 5, 6, and 8 remove second spellings and close ambiguities, and rows 5 to 24 below are that work. Everything from row 25 on is the inline layer, and that is where the break is.

## The block layer

| # | Delta | Decision | Translation |
|---|-------|----------|-------------|
| 1 | `$` is never math syntax; bare `$` is always literal text | 1 | None needed — text that was at risk becomes safe |
| 2 | Math is core: `` `x=y`{math} `` inline, ```` ```math ```` fenced | 1 | Rewrite renderer-specific dollar math |
| 3 | Setext headings removed | 4 | Convert underlined headings to ATX |
| 4 | Heading closing sequences removed; a trailing run of `#` is heading text | 4 | Migrator strips the closing run and reports it |
| 5 | Headings are never indented; the `#` sits in the first column | 4 | Outdent; nothing else changes |
| 6 | Empty headings removed; a heading needs text | 4 | A lone `#` line becomes a paragraph — migrator reports each one |
| 7 | Indented code blocks removed | 5 | Convert four-space blocks to fences |
| 8 | A verbatim fence closes on a run of **exactly** the opening length; a longer run alone on a line is content | 5 | Migrator shortens an over-long closing run and reports it |
| 9 | Strict list rules: content-column alignment, no lazy continuation | 6 | Re-indent continuations |
| 10 | `-` is the only bullet marker; `*` and `+` are not | 6 | Migrator rewrites `*` and `+` bullets to `-` |
| 11 | `1.` is the only ordered marker; `1)` is not | 6 | Migrator rewrites `1)` to `1.` |
| 12 | Adjacent items are one list; switching marker no longer separates two lists | 6 | No automatic path — migrator reports each pair it cannot separate |
| 13 | Raw HTML is removed entirely; there is no passthrough of any kind | 7 | No automatic path — migrator strips and reports; see decision 7 |
| 14 | Pipe tables are core and strictly grammatized | 8 | Malformed tables become paragraphs instead of being guessed at |
| 15 | Any line beginning with `\|` opens a table; leading and trailing pipes are mandatory | 8 | Migrator adds the pipes GFM let you omit |
| 16 | A rule-row cell is three or more `-`; a single hyphen no longer matches, and a rule row is read only as a table's first or second row | 8 | Migrator pads short rule rows; a dash cell elsewhere becomes content |
| 17 | A rule row may come first (no header) or be absent entirely (column count from the first row) | 8 | New capability; nothing to translate |
| 18 | Block cells: `\|` alone on a line opens a cell holding paragraphs, lists, or fences | 8 | New capability; nothing to translate |
| 19 | A table ends at a blank line after a line beginning with `\|`; a block-final table closes with a lone `\|` | 8 | None for inline tables — they end as they always did |
| 20 | Decorators `{word}` on fences and verbatim spans; labels are opaque; no classes | 9 | New capability; nothing to translate |
| 21 | Tabs are not structural | 10 | Validator warns and auto-fixes |
| 22 | No smart punctuation in core | 11 | Typographic quotes must be written literally |
| 23 | Unicode throughout, UTF-8 encoded; content is never normalized | 14 | None expected; see the open riders in decision 14 |
| 24 | The thematic break is removed; `---`, `***`, `___` are ordinary text | 19 | Migrator reports each one — a break is a heading that was not written, and only the author knows which |

## The inline layer

Everything below follows from one rule (decision 20): **a sigil carries meaning only when the very next code point is `{`.**

| # | Delta | Decision | Markdown | Markleft |
|---|-------|----------|----------|----------|
| 25 | Emphasis is braced; there is no paired form and no flanking rule | 2, 20 | `*em*` | `*{em}` |
| 26 | Strong emphasis is braced | 2, 20 | `**strong**` | `**{strong}` |
| 27 | Emphasis and strong together are one construct | 2, 20 | `***both***` | `***{both}` |
| 28 | Underscore is not emphasis syntax | 2 | `_em_` | `*{em}` — and `snake_case` becomes safe |
| 29 | Links are sigil-marked; the target is in braces | 20 | `[text](url)` | `@[text]{url}` |
| 30 | A link with no text renders its target | 20 | `<https://example.org>` | `@{https://example.org}` |
| 31 | Images are sigil-marked; the source is in braces | 16, 20 | `![alt](src)` | `![alt]{src}` |
| 32 | Anchors put the sigil before the brace | 17, 20 | *(no equivalent)* | `#{id}` |
| 33 | Reference links removed, with their definitions | 20 | `[text][label]` | `@[text]{url}` — written where it is used |
| 34 | Shortcut reference links removed | 20 | `[label]` | Ordinary text — `[sic]` is safe with no escape |
| 35 | Autolinks removed | 20 | `<https://x.org>` | `@{https://x.org}` |
| 36 | Escaping is a range, not a character | 3, 20 | `\|` | `\{\|}` |
| 37 | A single backslash escapes nothing; `\n` and `C:\Users` survive | 3, 20 | `\n` renders `\n` | `\n` renders `\n` — no longer a delta |
| 38 | Trailing-space hard breaks removed; `\` before a line ending is the only one | 13 | two trailing spaces | `\` at end of line |
| 39 | Superscript and subscript, braced always, typographic only | 18 | `<sup>2</sup>` | `x^{2}`, `H_{2}O` |

## Per-delta detail

Each row above becomes a subsection here, and each subsection needs the same four things:

- **What changed** — the CommonMark behaviour and the Markleft behaviour, stated as a pair.

- **Why** — which invariant or decision forces it. A delta that cannot name one is a delta with nothing holding it up.

- **What breaks** — the realistic input that renders differently, not the contrived one. This is what a reader came for.

- **What the migrator does** — automatic, automatic with a report, or manual.

## Notes for drafting

**Rows 25 to 39 are one story, and the file should say so before it lists them.** A reader who meets fifteen separate inline changes will read them as fifteen arbitrary choices. A reader who meets the rule first — *a sigil means nothing unless a brace follows* — meets one change with fifteen consequences, and can predict all fifteen from the rule. Lead with the rule, then the table.

**Row 37 is the entry to write with the most care, because it is the only row that runs backwards.** Every other delta asks a reader to change something. This one tells them that a thing which used to break now works: an earlier draft of this language escaped every character after a backslash, so `C:\Users\frederic` rendered as `C:Usersfrederic` and `\d+` lost its `d`. That was worse than CommonMark, which escapes ASCII punctuation only. Decision 20 removed it. The entry exists so that anyone who read the earlier specification learns the rule changed, and so that the failure is on the record rather than quietly absent.

**Row 34 is the largest prose-safety gain in the table and it looks like a removal.** Under CommonMark's shortcut form, `[sic]`, `[1]`, `[TODO]`, and `[citation needed]` become links whenever a definition with a matching label exists anywhere in the same document — including one added months later by someone who never saw the paragraph they changed. It was the only construct in either language whose meaning depended on text arbitrarily far away, and it failed silently when it fired. Write the entry around that, not around the syntax.

**Rows 29 and 31 should be written as one pair, because the mnemonic is the point.** `@[…]` links to the target; `![…]` shows it instead. The two differ by exactly one character, where in Markdown they differ by a bracket-and-parenthesis dance. A reader who learns the pair learns both.

**Row 24 is the row most likely to be argued with, so it has to lead with the reason rather than the rule.** The reason is not that `---` is ambiguous — it is not — but that it never said anything: a run of hyphens announces a change and never names it, where a heading names it. State the renderer collision second, because it is the one a reader recognizes immediately. And be honest that this is the one delta with no automatic migration: converting one needs a decision only the author can make.

**Row 39 is the only row that makes row 13 easier.** Raw HTML has no automatic migration path in general, and `<sup>` and `<sub>` were the two tags a Markdown author reached for most often with no alternative available — so this pair converts mechanically, and the migrator should say so rather than reporting them alongside the tags it genuinely cannot handle. The entry also has to carry the negative: there is no braceless form, so `x^2` migrated from `x<sup>2</sup>` is wrong and `x^{2}` is right. A braceless form would have to guess where the raised text stops, and guessing after one digit renders `2^32` as 2³2 — wrongly rather than partially.

**Row 8 is the smallest entry in the table and the easiest to under-explain.** It fires only on a closing fence written longer than the fence it closes, which nobody does deliberately. What the entry has to convey is the *reason*, because the reason is the whole counting story — the run length is the only escape available inside verbatim content, and making the match exact gives one counting rule at every size instead of two. Under decision 20 that rule now covers brace groups as well, so this entry is where a reader first meets a sentence that governs three constructs.

**Rows 10, 11, and 12 are the marker reduction, and they are the friendliest entries in the table:** 10 and 11 convert mechanically and losslessly, and 12 is the only one that cannot, since two lists separated by a marker switch cannot be told apart once the switch is gone.

**Rows 15 to 19 are the table rewrite, and they are unusual here for being mostly *additions*** — 17 and 18 take nothing away, and 19 changes nothing for a table written the way Markdown writes them. Only 15 and 16 make the migrator touch an existing document, and both do so mechanically. Write these so a reader sees that immediately, because "we rewrote tables" reads alarming until you learn that every table you already have converts without a judgement call.

**Row 16 is the one to explain rather than merely state, because the reason is a hazard rather than a preference.** GFM accepts a single hyphen in a rule-row cell, and gets away with it because a delimiter row can only ever be a table's second line. Once a rule row may also come first — row 17 — a headerless table opening `| - | - |` would parse as structure and **those two cells would disappear.** Silently losing content is the one failure this project treats as disqualifying, so the minimum went to three, which is what fences already ask for.

**Rows 3, 4, and 5 are the block layer's muscle-memory cluster**, flagged in the decision record as carrying the highest habit risk there. The inline layer now carries more than all of them together, which changes how the whole file should be framed: the question is no longer "which of these will trip a Markdown author" but "how long does it take to learn one rule". Write for the second question.
