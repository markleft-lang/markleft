# Memo: should verbatim use some character other than the backtick?

*Non-normative. Status: challenge raised and declined 2026-08-07. Decision 11 stands unchanged. Recorded per the working convention that a challenge to a settled decision is written down rather than resolved silently, so that a later session finds the analysis instead of repeating it.*

## The challenge

The backtick is hostile to people who did not grow up on Markdown, and to anyone off a US keyboard:

- It is visually close to `'` and `´`, and at small sizes or in some fonts they are hard to tell apart.

- It is a **dead key** on several common layouts. On German QWERTZ it is `Shift`+`´` followed by a space; on French AZERTY it is `AltGr`+`7`; Nordic and Spanish layouts have their own variants. A dead key produces nothing visible until the next keystroke, and then may produce an accented letter instead. That is genuine hostility, not a matter of taste.

- Mobile keyboards bury it two or three layers deep.

Proposed alternative: the pipe, as `|x=y|{math}`.

## Why the pipe fails, specifically

Three counts, of which the first two are fatal on their own.

**It is already the table delimiter.** Decision 8 puts pipe tables in the core. Giving one character two structural jobs is precisely the disease this language exists to cure, and it would be introduced deliberately rather than inherited.

**It breaks prose-safety, which is invariant 1.** People write `|x| < 5` for absolute value, `P(A|B)` for conditional probability, `a | b` when discussing shell pipelines or BNF alternation. Every one of those becomes a verbatim span. This is the `$` failure exactly: a character that occurs naturally in prose, silently promoted to a delimiter. The project's founding exhibit is what happens next.

**It costs the compatibility budget.** Decision 12 matches CommonMark byte-for-byte wherever that is unambiguous and harmless, and backtick verbatim is both. Changing it also ends self-hosting stage 1 — every `.md` file in these repositories stops rendering correctly on GitHub, the platform whose choice decided Markdown's adoption in the first place.

## Why no other character works either

The unclaimed ASCII, surveyed: `"` and `'` occur constantly in prose; `~` is GFM strikethrough and is also a dead key on some layouts; `^` is a dead key on *more* layouts than the backtick, and is exponentiation besides; `%`, `!`, `:`, and `/` are ordinary prose and dates and paths; `#`, `\`, `*`, and `_` are already spent; `@` is handles and addresses. Nothing survives.

That is not bad luck, and this is the part worth keeping:

**The backtick's inaccessibility and its prose-safety are the same property.** It is hard to reach because almost nobody types it in ordinary writing — and almost nobody typing it in ordinary writing is exactly what qualifies it as a delimiter. Any character reachable enough to fix the keyboard complaint is a character common enough to re-create the collision. The trade is structural rather than a historical accident of Gruber's, which means no amount of searching produces a candidate that wins on both axes.

A second delimiter *alongside* the backtick is not an escape from this either: it costs the "one way to do it" discipline and the five-minute property, for a construct that is not broken.

## Verdict, and where the concern is answered instead

Keep the backtick. The concern is real but it is a **tooling** problem, and it gets tooling answers — the same place tables of contents went under decision 15:

- **Editor support.** Snippets and keybindings, which every editor already has, and which cost the language nothing.

- **The cheat sheet** (Phase 4) states how to type a backtick on the common non-US layouts. One line of documentation against a recurring papercut is a good trade.

- **A validator diagnostic** (Phase 3), which is the sharpest available answer. A decorator that follows something which is not a verbatim span is almost always a mis-typed backtick, and the validator can say so: *"decorator with no verbatim span — did you type `'` or `´` where a backtick was meant?"*. Lookalike detection at the point of failure turns the confusion into a fixable message rather than a silently wrong document. Typst's error-message bar is recorded in `.claude/landscape.md` as the standard to meet, and this is exactly the sort of case it is meant for.

## What would reopen this

Evidence, not argument: corpus or user data showing that layout friction actually blocks adoption, **together with** a candidate character that survives the prose-safety test above. Absent a candidate, the challenge has nowhere to go — the problem is well understood and the alternatives are worse.
