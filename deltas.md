# Deltas from Markdown

*Crown copyright — to be released CC0 pending review.*

**Status: scaffold.** The delta list below is complete as to *which* decisions diverge from CommonMark, because those decisions are settled. The per-delta detail — exact before and after, edge cases, migrator behaviour — is not written yet, and **no text in this file is normative until it is.**

This document is Appendix A of `charter.md`, and it has two audiences at once. For an implementer it is the normative list of every point where Markleft departs from CommonMark. For someone moving documents across, it is the migration guide. Decision 12 requires that every divergence appear here: matching CommonMark byte-for-byte wherever that is unambiguous and harmless is the rule, so each entry below has to justify its own existence.

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
| 10 | Strict list rules: one marker per list, content-column alignment, no lazy continuation | 6 | Re-indent continuations; split mixed-marker lists |
| 11 | Raw HTML is removed entirely; there is no passthrough of any kind | 7 | No automatic path — migrator strips and reports; see decision 7 |
| 12 | Pipe tables are core and strictly grammatized | 8 | Malformed tables now fail instead of guessing |
| 13 | Decorators `{word .class #id}` on fences and verbatim spans — the only extension point | 9 | New capability; nothing to migrate |
| 14 | Tabs are not structural | 10 | Validator warns and auto-fixes |
| 15 | No smart punctuation in core | 11 | Typographic quotes must be written literally |
| 16 | Trailing-space hard breaks removed; `\` before a line ending is the only hard break | 13 | Migrator converts and reports each one |
| 17 | Unicode throughout, UTF-8 encoded; content is never normalized | 14 | None expected; see the open riders in decision 14 |

## Per-delta detail

Each row above becomes a subsection here, and each subsection needs the same four things:

- **What changed** — the CommonMark behaviour and the Markleft behaviour, stated as a pair.

- **Why** — which invariant or decision forces it. A delta that cannot name one is a delta that should not exist, per decision 12.

- **What breaks** — the realistic input that renders differently, not the contrived one. This is what a reader came for.

- **What the migrator does** — automatic, automatic with a report, or manual. Delta 16 is the case to get right first: the old syntax is invisible, so a reader cannot audit that conversion by eye and has to trust the report.

## Notes for drafting

Delta 4 deserves care. Removing the exception list makes escaping uniform, but it also means a backslash that used to survive as a literal character now consumes the character after it. That is a silent change in output rather than a parse error, which makes it the most dangerous entry in this table for anyone porting documents — the opposite of delta 13, which is invisible in the source but noisy in the result.

Deltas 5, 6, and 7 are the muscle-memory cluster. They are the three the decision record flags as carrying the highest risk, and the corpus that would have measured their real-world impact was dropped. Expect the migrator's change report to be the first real evidence about them, which is late; write these entries so that evidence is easy to act on when it arrives.
