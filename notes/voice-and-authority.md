# Memo: whose authority is a rule speaking with?

*Non-normative. Status: decided 2026-08-08. Records a language pass over every document in the project, and the test that produced it, so that later sessions write in the same register instead of rediscovering it. House-style material: the substitution corpus below is the working part.*

## The problem

Markleft is written to be community-owned. Nothing about the project's substance stands in the way of that — the invariants are shared commitments, the decision record exists so a stranger can argue with it, and the licence exists so nobody needs permission. But the *prose* had drifted into a register that says something else. A reader of `definition.md` alone met the word "forbid" fifteen times, "rejected regardless of merit" twice, and fifteen bold **Rejected** headers, before reaching a single sentence about how to disagree.

That register is not what the project means. It crept in for an ordinary reason: a specification is a document full of rules, and rules attract the vocabulary of authority even when no authority is doing anything. "Invariant 2 forbids an exception list" is not a prohibition issued by anyone. It is a statement that invariant 2 and an exception list cannot both be true.

The distinction that matters is not strong versus soft. It is **objective versus institutional**.

## The test

Whenever an authoritative word appears, ask **who or what is doing the ruling.** Three answers, three treatments.

**1. The grammar, a program, or a definition is doing it.** Keep the authority — and notice that saying it directly usually makes the sentence shorter and more precise. `cannot`, `does not match`, `is text`, `has exactly one parse`, `never fails`, `must` on a conforming implementation. These are checkable by the conformance suite, which is the whole point: a machine settles them, not a person. This is the register the definition should be *densest* in.

**2. A rule is being personified as an authority.** Say what is actually true of the rule instead. This is where most of the drift lived, and the fix is nearly always an improvement in accuracy: "invariant 2 forbids those" becomes "invariant 2 leaves no room for those", which is the real relationship.

**3. A person or the project is doing it.** Reframe as consequence, cost, or the path to change it. This is the only category where the authority claim is genuinely a claim, and it is the one to give up. The project already has the canonical move, in `.claude/decisions.md` §2: **the answer is not *no*; it is *not in the format*** — and the burden then moves to showing the thing cannot be written into the source by a tool. That sentence is the whole voice in miniature.

## The corpus

### Where a rule was personified

| Authority phrasing | What is actually true | Open phrasing |
|---|---|---|
| invariant 2 forbids those | an exception list and invariant 2 cannot both hold | invariant 2 leaves no room for those |
| which invariant 4 forbids | lookahead and prefix-decidability are incompatible | which invariant 4 rules out |
| the clause invariant 2 exists to forbid | | the clause invariant 2 was written against |
| decision 15 forbids it | the construct falls outside the rule's boundary | decision 15 leaves it out; is where decision 15 stops |
| decision 12 forbids diverging for legibility | decision 12 sets a bar the argument does not clear | under decision 12 a divergence needs more than legibility |
| what the rule forbids is | | what the rule excludes is |
| invariant 2 refuses | | invariant 2 has no room for |
| Markdown's lack of styling softly enforces uniformity | | quietly produces uniformity |
| each is enforced by a rule | | each is held by a rule |
| file includes stay forbidden | | file includes stay out of the language |
| the prohibition is over-determined | | the case is over-determined |

### Where the project spoke as an authority over people

| Authority phrasing | Open phrasing |
|---|---|
| a feature that violates one is rejected regardless of merit | a proposal that breaks one cannot land as it stands — changing that means amending the invariant itself, in the open, with the argument written down |
| is declined regardless of merit | does not clear the bar, however much it offers elsewhere |
| a format whose output is indistinguishable is refused | does not clear the design test |
| better handled by saying no | better carried in the tooling |
| the refusals so far have been cheap | the answers so far have been cheap |
| Reopening this requires a candidate character | what would reopen this is a candidate character |
| Vendors are advised to prefix `x-` | prefixing `x-` is the usual guard; nothing checks it |
| so a contributor does not relitigate a settled question | so a contributor finds the analysis instead of rebuilding it |
| this does not license the next request | the next case is read on its own terms rather than inheriting this one |
| **Rejected —** *(as a decision-record heading)* | **Not adopted —** |
| the alternatives that were rejected | the alternatives it set aside |

The last two are the closest call in this memo. "Rejected alternatives" is standard decision-record vocabulary and records a true fact. It went anyway, because thirty bold **Rejected** headers read as verdicts handed down on the people who proposed them, and "not adopted" records the identical fact while leaving the door visibly open — which is the accurate state of every one of them.

### Where a permission word stood in for a fact

| Authority phrasing | Open phrasing |
|---|---|
| problems a parser is not allowed to fail on | problems a parser never fails on |
| the grammar permits it anyway | the grammar has room for it anyway |
| it may never supply text that is not | and never supplies text that is not |
| only marks may be consumed | only marks are consumed |
| no behaviour may exist only in the suite | no behaviour exists only in the suite |
| empty alternative text is legal | empty alternative text is well-formed |
| CommonMark allowed up to three spaces | CommonMark admitted up to three spaces |
| that restriction exists for the same reason | that narrowing exists for the same reason |
| A reader must be able to read the standard alone | a reader can read the standard alone |
| the language must outlive its authors | the language should be able to outlive its authors |

Prefer **`cannot`** over **`may not`** wherever the sentence is about the grammar. One is a fact about the language; the other is a permission somebody withheld. They are rarely both available, and when they are, the fact is the better sentence.

### The vocabulary that was already right

Most of the project was already in the open register, and the pass was about the minority that slipped. This is the house voice, and it is worth naming so it gets reached for first: *earns its place, pays for itself, buys, costs, is free, enabling not restrictive, relocated not forbidden, clears the bar, survives the test, has nowhere to go, nobody needs permission to test, the answer is not no — it is not in the format.*

Note what these have in common. They are all **economic rather than juridical**: a construct pays, earns, buys, or costs, and the reader can check the arithmetic. That is the same move as making the specification executable — it hands the judgement to something a stranger can run rather than to somebody they have to trust.

## What keeps its authority, deliberately

Softening these would cost precision for nothing, and every one of them is settled by a machine rather than by a person:

- ***Must*, *must not*, and *may* on a conforming implementation.** Defined in the definition's requirement-words paragraph, checkable by the conformance suite. The definition uses no *should*, and that stands: a rule that only recommends is a rule the language cannot rely on.

- **"The document governs."** A routing rule between two artefacts, and the definition already says so in as many words — it settles *where a change must land*, not *who was right*.

- **"Never", "always", "exactly one".** Quantifiers over inputs. `$` is never syntax; every input has exactly one parse. These are the guarantees, and hedging them would be a lie.

- **"Strict", "exact", "closed", "inert by construction".** Properties of a grammar, not dispositions of a maintainer.

- **The corpus and licensing constraints in `tests`.** "Nothing enters `corpus/` that we did not write" is a licence-integrity fact with a checkable boundary, not a house rule about conduct.

## Where this applies

Every document in both repositories, and every one written from here. The reference to reach for when writing new normative prose is the test above: *who is doing the ruling.* If the answer is the grammar, say so plainly and keep the force. If the answer is us, say what it costs and how it would change.
