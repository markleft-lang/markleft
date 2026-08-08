# Memo: should the rule row carry column width hints?

*Non-normative. Status: challenge raised and declined 2026-08-08. Decision 8 stands without width hints. Recorded per the working convention that a challenge to a settled position is written down rather than resolved silently — and in this case because the challenge was partly right, and three of the arguments first made against it are withdrawn here rather than left standing.*

## The proposal

Column widths written into the rule row, alongside the alignment colons that are already there: `|70:-----|30-----|`, the numbers being a percentage of the table's total width.

It addresses a genuine frustration. A table with one prose column and three short ones renders badly almost everywhere, because automatic sizing has no way to know which column deserves the room. The author knows and cannot say so.

## What was wrong with the first refusal

Three arguments were offered against it and all three fail. They are recorded because leaving a bad argument in the record is how a good decision gets overturned later by someone who notices.

**"A percentage has no meaning in plain text, because there is no canvas."** False. The canvas is the source line, and the canonical formatter already has to choose column widths in order to pad cells at all. If it can express alignment in the source — and editors already do exactly this, padding cells so text sits left, right, or centred — then it can express proportions the same way, approximately. So a width hint *would* be visibly distinguished in plain text, which is the test decision 15's back half sets. This was the strongest-sounding argument and it is simply not true.

**"It is page or layout awareness, which §2 rules out."** Weak. A proportion carries no page geometry and no absolute measure. It is medium-independent in a way that inches, points, and pixels are not, and the non-goal was written against document-preparation systems that know about pages.

**"It reintroduces the founding exhibit — same source, different documents."** Backwards. A stated proportion is *more* deterministic across renderers than automatic sizing, which varies with font metrics and content measurement. And width does not change what the text says, so the founding exhibit was never the right analogy.

## What actually decides it

**A width is a number maintained by hand, and it goes stale silently.** Content grows, a column that wanted seventy per cent now wants forty-five, and nothing says so — the table simply renders worse until somebody notices and re-tunes it. That is the same shape this project has already rejected three times: hand-maintained ordered-list numerals (decision 6), numbered rather than named anchors (decision 17), and hard-wrapped prose (`.claude/markdown-conventions.md`). Each one is a value written into the source that has to be maintained against something that moves, and each fails invisibly.

The sharper form of it, and the one that settled the question: **a number in the source would actively fight whatever sizing code someone writes later.** Automatic sizing is a formatter and renderer problem, and it is a solvable one — measuring content and distributing width well is ordinary engineering that will keep improving. A hint in the source is a permanent override of every future improvement, applied by an author who was looking at one version of one table in one medium.

Two further counts, neither of which would have been enough alone:

**The continuity is what costs.** Alignment cannot be tuned — it has three values and an author picks one in a second. A percentage invites adjustment, then re-adjustment, and that is precisely the "pixel-perfect subjective loop" that decision 9 gave as the deciding reason for removing `.class`. Being freed from styling decisions is a feature rather than a deprivation, and a continuous layout parameter hands the decision back.

**There is nowhere for it to land.** Decorators attach only to verbatim constructs and carry no key-value pairs, so a width cannot arrive through the language's extension point; it would have to grow the table grammar itself. That the back door is closed is decision 15 working as intended, and it is evidence the language is internally consistent rather than an argument on its own.

## Why alignment survives and width does not

The parallel is fair and has to be answered rather than dodged: alignment is presentational too. It is. The distinction is not that one is styling and the other is not — it is that one is **closed, three-valued, and describes the content**, and the other is **open, continuous, and describes only the display**.

Right alignment says *these are quantities*. Centring says *these are short labels*. A reader can apply either in their head, and a plain-text reader who ignores both loses nothing. Seventy per cent says nothing about what the column contains; it is an instruction to a layout engine and to nothing else.

Alignment also keeps GFM tables working unchanged, which matters under decision 12's spirit even though pipe tables are not in CommonMark and therefore not protected by it.

## Where the want goes instead

It splits in two, and most of it is already handled.

**Source readability with lopsided columns** — the larger half of the frustration in practice — is answered by block cells (decision 8, 2026-08-08). A long cell no longer forces a long line, so the source stays readable without anyone stating a width.

**A specific rendered proportion** goes where §2 sends every capability of this kind: downstream, to a stylesheet or a template in the publishing pipeline. That is not a hardship for anyone publishing a table where proportion genuinely matters, and it costs the plain-text reader nothing.

**Source column padding** is the only width that exists in plain text, and it belongs to the canonical formatter (Phase 3). If an author wants a wide first column in the source, that is a formatter setting, and a formatter setting can be shared alongside a document without the language carrying it forever.

## What would reopen this

Evidence, and a spelling — the same bar as `notes/backtick-verbatim-challenge.md`:

- Evidence that renderer and formatter automatic sizing fails often enough in practice that a hand-maintained number is worth its staleness. Corpus or user data, not intuition.

- **And** a spelling that a plain-text formatter can express faithfully, so the mark is not consumed with nothing replacing it on platforms that strip inline styles.

Without both, the challenge has nowhere to go: the want is real, the mechanism is available, and the reason for refusing is that the number will be wrong later and nobody will be told.
