# Deltas from Markdown

*Crown copyright, Canada — 2026. Licensed CC BY 4.0.*

**Status: scaffold.** The delta list below is complete as to *which* decisions diverge from CommonMark, because those decisions are settled. The per-delta detail — exact before and after, edge cases, migrator behaviour — is not written yet, and **no text in this file is normative until it is.**

**Row numbers here are not stable identifiers.** They renumber whenever a delta is inserted, which has happened four times already, so cite the *decision* number when referring to a divergence anywhere outside this file. Decision numbers never move; these do.

This document is Appendix A of `definition.md`, and it has two audiences at once. For an implementer it is the normative list of every point where Markleft departs from CommonMark. For someone moving documents across, it is the migration guide. Under decision 12 every divergence appears here: matching CommonMark byte-for-byte wherever that is unambiguous and harmless is the rule, so each entry below has to justify its own existence.

## The deltas

| # | Delta | Decision | Migration |
|---|-------|----------|-----------|
| 1 | `$` is never math syntax; bare `$` is always literal text | 1 | None needed — text that was at risk becomes safe |
| 2 | Math is core: `` `x=y`{math} `` inline, ```` ```math ```` fenced | 1 | Rewrite renderer-specific dollar math |
| 3 | Underscore is not emphasis syntax | 2 | `_em_` becomes `*em*`; `snake_case` becomes safe |
| 4 | Backslash escapes **any** character, with no exception list | 3 | Escapes that were literal backslashes now escape |
| 5 | Setext headings removed | 4 | Convert underlined headings to ATX |
| 6 | Heading closing sequences removed; a trailing run of `#` is heading text | 4 | Migrator strips the closing run and reports it |
| 7 | Headings are never indented; the `#` sits in the first column | 4 | Outdent; nothing else changes |
| 8 | Empty headings removed; a heading needs text | 4 | A lone `#` line becomes a paragraph — migrator reports each one |
| 9 | Indented code blocks removed | 5 | Convert four-space blocks to fences |
| 10 | A verbatim fence closes on a run of **exactly** the opening length; a longer run alone on a line is content | 5 | Migrator shortens an over-long closing run and reports it |
| 11 | Strict list rules: content-column alignment, no lazy continuation | 6 | Re-indent continuations |
| 12 | `-` is the only bullet marker; `*` and `+` are not | 6 | Migrator rewrites `*` and `+` bullets to `-` |
| 13 | `1.` is the only ordered marker; `1)` is not | 6 | Migrator rewrites `1)` to `1.` |
| 14 | Adjacent items are one list; switching marker no longer separates two lists | 6 | No automatic path — migrator reports each pair it cannot separate |
| 15 | Raw HTML is removed entirely; there is no passthrough of any kind | 7 | No automatic path — migrator strips and reports; see decision 7 |
| 16 | Pipe tables are core and strictly grammatized | 8 | Malformed tables become paragraphs instead of being guessed at |
| 17 | Any line beginning with `\|` opens a table; leading and trailing pipes are mandatory | 8 | Migrator adds the pipes GFM let you omit |
| 18 | A rule-row cell is three or more `-`; a single hyphen no longer matches, and a rule row is read only as a table's first or second row | 8 | Migrator pads short rule rows; a dash cell elsewhere becomes content |
| 19 | A rule row may come first (no header) or be absent entirely (column count from the first row) | 8 | New capability; nothing to migrate |
| 20 | Block cells: `\|` alone on a line opens a cell holding paragraphs, lists, or fences | 8 | New capability; nothing to migrate |
| 21 | A table ends at a blank line after a line beginning with `\|`; a block-final table closes with a lone `\|` | 8 | None for inline tables — they end as they always did |
| 22 | Decorators `{word}` on fences and verbatim spans; labels are opaque; no classes | 9 | New capability; nothing to migrate |
| 23 | Tabs are not structural | 10 | Validator warns and auto-fixes |
| 24 | No smart punctuation in core | 11 | Typographic quotes must be written literally |
| 25 | Trailing-space hard breaks removed; `\` before a line ending is the only hard break | 13 | Migrator converts and reports each one |
| 26 | Unicode throughout, UTF-8 encoded; content is never normalized | 14 | None expected; see the open riders in decision 14 |
| 27 | Positional anchors `{#id}` valid anywhere; references stay ordinary links | 17 | New capability; nothing to migrate |
| 28 | Superscript `^{…}` and subscript `_{…}`, braced always, typographic only | 18 | New capability — and the one automatic path out of delta 15: `<sup>` and `<sub>` convert |
| 29 | The thematic break is removed; `---`, `***`, `___` are ordinary text | 19 | Migrator reports each one — a break is a heading that was not written, and only the author knows which |

## Per-delta detail

Each row above becomes a subsection here, and each subsection needs the same four things:

- **What changed** — the CommonMark behaviour and the Markleft behaviour, stated as a pair.

- **Why** — which invariant or decision forces it. A delta that cannot name one is a delta with nothing holding it up, per decision 12.

- **What breaks** — the realistic input that renders differently, not the contrived one. This is what a reader came for.

- **What the migrator does** — automatic, automatic with a report, or manual. Delta 25 is the case to get right first: the old syntax is invisible, so a reader cannot audit that conversion by eye and has to trust the report.

## Notes for drafting

Delta 4 deserves care. Removing the exception list makes escaping uniform, but it also means a backslash that used to survive as a literal character now consumes the character after it. That is a silent change in output rather than a parse error, which makes it the most dangerous entry in this table for anyone porting documents — the opposite of delta 25, which is invisible in the source but noisy in the result.

Delta 29 is the row most likely to be argued with, so it has to lead with the reason rather than the rule. The reason is not that `---` is ambiguous — it is not — but that it never said anything: a run of hyphens announces a change and never names it, where a heading names it. State the renderer collision second, because it is the one a reader recognizes immediately — most house styles already draw a thin rule under a level-2 heading, so a document using both gets two. And be honest that this is the one delta with no automatic migration: the migrator finds every break and reports it, because converting one needs a decision only the author can make, between a heading and a second document.

Delta 28 is the only row in the table that makes delta 15 *easier*. Raw HTML has no automatic migration path in general, and `<sup>` and `<sub>` were the two tags a Markdown author reached for most often with no alternative available — so this pair converts mechanically, and the migrator should say so rather than reporting them alongside the tags it genuinely cannot handle. The entry also has to carry the negative, because that is what a reader arriving from HTML will test first: there is no braceless form, so `x^2` migrated from `x<sup>2</sup>` is wrong and `x^{2}` is right. State the reason next to it — a braceless form would have to guess where the raised text stops, and guessing after one digit renders `2^32` as 2³2, wrongly rather than partially.

Delta 10 is the smallest entry in the table and the easiest to under-explain. Almost no document is affected: it fires only on a closing fence written longer than the fence it closes, which nobody does deliberately. What the entry has to convey is the *reason*, because the reason is the whole counting story — the run length is the only escape available inside verbatim content, and making the match exact gives one counting rule at both sizes instead of two. State the failure mode plainly alongside it: an over-long closing run no longer closes, so the block runs to the end of its container, and the validator is what catches it.

Deltas 12, 13, and 14 are the marker reduction, and they are the friendliest entries in the table: 12 and 13 convert mechanically and losslessly, and 14 is the only one that cannot, since two lists separated by a marker switch cannot be told apart once the switch is gone. The migrator reports those rather than merging them silently.

Deltas 17 to 21 are the table rewrite, and they are unusual in this table for being mostly *additions* — 19 and 20 take nothing away, and 21 changes nothing at all for a table written the way Markdown writes them. Only 17 and 18 make the migrator touch an existing document, and both do so mechanically: adding the pipes GFM let you omit, and padding a rule row that used fewer than three hyphens. Write these entries so a reader sees that immediately, because "we rewrote tables" reads alarming until you learn that every table you already have converts without a judgement call.

Delta 18 is the one to explain rather than merely state, because the reason is a hazard rather than a preference. GFM accepts a single hyphen in a rule-row cell, and it gets away with it because a delimiter row can only ever be a table's second line. Once a rule row may also come first — delta 19 — a headerless table opening `| - | - |` would parse as structure and **those two cells would disappear.** Silently losing content is the one failure this project treats as disqualifying, so the minimum went to three, which is what fences already ask for. *(Thematic breaks asked for three as well, until delta 29 removed them; the fence minimum still carries the argument on its own.)*

Deltas 5, 6, and 7 are the muscle-memory cluster. They are the three the decision record flags as carrying the highest risk, and the corpus that would have measured their real-world impact was dropped. Expect the migrator's change report to be the first real evidence about them, which is late; write these entries so that evidence is easy to act on when it arrives.
