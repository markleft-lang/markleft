# Memo: should verbatim use some character other than the backtick?

*Non-normative. Status: challenge raised and declined 2026-08-07. Decision 11 stands unchanged. Recorded per the working convention that a challenge to a settled decision is written down rather than resolved silently, so that a later session finds the analysis instead of rebuilding it.*

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

**It costs the compatibility budget.** Decision 12 matched CommonMark byte-for-byte wherever that was unambiguous and harmless, and backtick verbatim is both.

*Updated 2026-08-09.* Decision 20 replaced the inline layer and decision 12 with it, so this third count no longer stands as written — compatibility is not a budget any more. **The first two counts are untouched and either is fatal on its own**, which is why the verdict does not move. What the change does is make the surviving argument *stronger*: backtick verbatim is now the only inline construct Markleft keeps from Markdown, and it is kept because it is right rather than because it was familiar. Decision 20's own reasoning says why the paired form is forced here and nowhere else — verbatim content is not interpreted, so its delimiter cannot be balance-counted, and run-length matching is the only close that works without reading the content.

## Why no other character works either

The unclaimed ASCII, surveyed: `"` and `'` occur constantly in prose; `~` is GFM strikethrough and is also a dead key on some layouts; `%`, `!`, `:`, and `/` are ordinary prose and dates and paths; `#`, `\`, `*`, `_`, and `^` are already spent; `@` is handles and addresses. Nothing survives.

*Updated 2026-08-08.* `^` moved from the unclaimed list to the spent one. This survey had rejected it on two grounds — a dead key on *more* layouts than the backtick, and exponentiation besides — and decision 18 then took it for superscript, which is the second of those two reasons arriving as a construct. The survey's conclusion is unchanged and its margin is one character narrower.

*Updated 2026-08-09.* `@` moved the same way, to decision 20's link. The survey had rejected it for colliding with handles and addresses, and the doublet rule answered that objection rather than overruling it: `@` alone is still free, and only `@{` and `@[` carry meaning. **That is the pattern worth extracting from two updates in two days.** Each character this survey rejected was rejected for occurring in prose, and the doublet rule makes occurring-in-prose stop mattering — what a construct needs is not an unused character but an unused *pair*. The survey's conclusion still holds for the backtick, because a verbatim delimiter cannot take a brace: its content is uninterpreted, so the close must be found by counting rather than by matching a scope. The one construct that cannot use the escape hatch is the one this memo is about.

That is not bad luck, and this is the part worth keeping:

**The backtick's inaccessibility and its prose-safety are the same property.** It is hard to reach because almost nobody types it in ordinary writing — and almost nobody typing it in ordinary writing is exactly what qualifies it as a delimiter. Any character reachable enough to fix the keyboard complaint is a character common enough to re-create the collision. The trade is structural rather than a historical accident of Gruber's, which means no amount of searching produces a candidate that wins on both axes.

A second delimiter *alongside* the backtick is not an escape from this either: it costs the "one way to do it" discipline and the five-minute property, for a construct that is not broken.

## Where the concern is answered instead

Keep the backtick. The concern is real but it is a **tooling** problem, and it gets tooling answers — the same place tables of contents went under decision 15:

- **Editor support.** Snippets and keybindings, which every editor already has, and which cost the language nothing.

- **The cheat sheet** (Phase 4) states how to type a backtick on the common non-US layouts. One line of documentation against a recurring papercut is a good trade.

- **A validator diagnostic** (Phase 3), which is the sharpest available answer. A decorator that follows something which is not a verbatim span is almost always a mis-typed backtick, and the validator can say so: *"decorator with no verbatim span — did you type `'` or `´` where a backtick was meant?"*. Lookalike detection at the point of failure turns the confusion into a fixable message rather than a silently wrong document. Typst's error-message bar is recorded in `.claude/landscape.md` as the standard to meet, and this is exactly the sort of case it is meant for.

## What would reopen this

Evidence, not argument: corpus or user data showing that layout friction actually blocks adoption, **together with** a candidate character that survives the prose-safety test above. Absent a candidate, the challenge has nowhere to go — the problem is well understood and the alternatives are worse.
