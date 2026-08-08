# Deltas from Markdown

*Crown copyright, Canada — 2026. Licensed CC BY 4.0.*

**Status: scaffold.** The delta list below is complete as to *which* decisions diverge from CommonMark, because those decisions are settled. The per-delta detail — exact before and after, edge cases, migrator behaviour — is not written yet, and **no text in this file is normative until it is.**

**Row numbers here are not stable identifiers.** They renumber whenever a delta is inserted, which has happened three times already, so cite the *decision* number when referring to a divergence anywhere outside this file. Decision numbers never move; these do.

This document is Appendix A of `definition.md`, and it has two audiences at once. For an implementer it is the normative list of every point where Markleft departs from CommonMark. For someone moving documents across, it is the migration guide. Decision 12 requires that every divergence appear here: matching CommonMark byte-for-byte wherever that is unambiguous and harmless is the rule, so each entry below has to justify its own existence.

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
| 10 | Strict list rules: content-column alignment, no lazy continuation | 6 | Re-indent continuations |
| 11 | `-` is the only bullet marker; `*` and `+` are not | 6 | Migrator rewrites `*` and `+` bullets to `-` |
| 12 | `1.` is the only ordered marker; `1)` is not | 6 | Migrator rewrites `1)` to `1.` |
| 13 | Adjacent items are one list; switching marker no longer separates two lists | 6 | No automatic path — migrator reports each pair it cannot separate |
| 14 | Raw HTML is removed entirely; there is no passthrough of any kind | 7 | No automatic path — migrator strips and reports; see decision 7 |
| 15 | Pipe tables are core and strictly grammatized | 8 | Malformed tables become paragraphs instead of being guessed at |
| 16 | Any line beginning with `\|` opens a table; leading and trailing pipes are mandatory | 8 | Migrator adds the pipes GFM let you omit |
| 17 | A rule-row cell is three or more `-`; a single hyphen no longer matches, and a rule row is read only as a table's first or second row | 8 | Migrator pads short rule rows; a dash cell elsewhere becomes content |
| 18 | A rule row may come first (no header) or be absent entirely (column count from the first row) | 8 | New capability; nothing to migrate |
| 19 | Block cells: `\|` alone on a line opens a cell holding paragraphs, lists, or fences | 8 | New capability; nothing to migrate |
| 20 | A table ends at a blank line after a line beginning with `\|`; a block-final table closes with a lone `\|` | 8 | None for inline tables — they end as they always did |
| 21 | Decorators `{word}` on fences and verbatim spans; labels are opaque; no classes | 9 | New capability; nothing to migrate |
| 22 | Tabs are not structural | 10 | Validator warns and auto-fixes |
| 23 | No smart punctuation in core | 11 | Typographic quotes must be written literally |
| 24 | Trailing-space hard breaks removed; `\` before a line ending is the only hard break | 13 | Migrator converts and reports each one |
| 25 | Unicode throughout, UTF-8 encoded; content is never normalized | 14 | None expected; see the open riders in decision 14 |
| 26 | Positional anchors `{#id}` valid anywhere; references stay ordinary links | 17 | New capability; nothing to migrate |

## Per-delta detail

Each row above becomes a subsection here, and each subsection needs the same four things:

- **What changed** — the CommonMark behaviour and the Markleft behaviour, stated as a pair.

- **Why** — which invariant or decision forces it. A delta that cannot name one is a delta that should not exist, per decision 12.

- **What breaks** — the realistic input that renders differently, not the contrived one. This is what a reader came for.

- **What the migrator does** — automatic, automatic with a report, or manual. Delta 24 is the case to get right first: the old syntax is invisible, so a reader cannot audit that conversion by eye and has to trust the report.

## Notes for drafting

Delta 4 deserves care. Removing the exception list makes escaping uniform, but it also means a backslash that used to survive as a literal character now consumes the character after it. That is a silent change in output rather than a parse error, which makes it the most dangerous entry in this table for anyone porting documents — the opposite of delta 24, which is invisible in the source but noisy in the result.

Deltas 11, 12, and 13 are the marker reduction, and they are the friendliest entries in the table: 11 and 12 convert mechanically and losslessly, and 13 is the only one that cannot, since two lists separated by a marker switch cannot be told apart once the switch is gone. The migrator reports those rather than merging them silently.

Deltas 16 to 20 are the table rewrite, and they are unusual in this table for being mostly *additions* — 18 and 19 take nothing away, and 20 changes nothing at all for a table written the way Markdown writes them. Only 16 and 17 make the migrator touch an existing document, and both do so mechanically: adding the pipes GFM let you omit, and padding a rule row that used fewer than three hyphens. Write these entries so a reader sees that immediately, because "we rewrote tables" reads alarming until you learn that every table you already have converts without a judgement call.

Delta 17 is the one to explain rather than merely state, because the reason is a hazard rather than a preference. GFM accepts a single hyphen in a rule-row cell, and it gets away with it because a delimiter row can only ever be a table's second line. Once a rule row may also come first — delta 18 — a headerless table opening `| - | - |` would parse as structure and **those two cells would disappear.** Silently losing content is the one failure this project treats as disqualifying, so the minimum went to three, which is what fences and thematic breaks already require.

Deltas 5, 6, and 7 are the muscle-memory cluster. They are the three the decision record flags as carrying the highest risk, and the corpus that would have measured their real-world impact was dropped. Expect the migrator's change report to be the first real evidence about them, which is late; write these entries so that evidence is easy to act on when it arrives.
