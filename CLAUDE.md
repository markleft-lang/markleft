# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

It holds operating guidance only — how to work here, and where everything lives. The project itself (what the language is, what has been decided, and why) is documented in the files it points to and is not summarized here: a summary is a copy, and copies drift.

## The project, in one paragraph

**Markleft** is a formally specified, ambiguity-free successor to Markdown — an independent project maintained by its authors, with no institutional steward, sponsor, or endorser. This repository is the standard itself: everything in it is normative except `CLAUDE.md` and `.claude/`, which are working scaffolding. Start with `README.md` for what the language promises and where the project stands.

## Where things live

- `README.md` — the pitch, the guarantees, the normative boundary, current status.

- `definition.md` and `deltas.md` — the normative documents (`grammar/` arrives at Phase 1).

- `.claude/decisions.md` — **the decision record**: invariants (§3), binding decisions (§4), name (§5), extensions (§6), licensing (§7), roadmap (§9), open checklist (§10), amendments (§11). **Read it before proposing any design change**, and cite decisions by number.

- `.claude/grammar.md` — the running rule list. Its reserved-position table is first in the file deliberately — the at-a-glance answer to "what is safe to type"; check any proposed decision against it before recording.

- `.claude/plan.md` — the running plan: current work, parked ideas, errands, and open questions, in plain language. Add to it as things come up.

- `.claude/landscape.md` — the prior-art survey; consult it when evaluating features other languages have.

- `notes/` — non-normative memos. `repo-layout.md` holds the three-repo organization structure and the cross-repo workflow; the sibling repos `../tests` and `../.github` carry their own thin `CLAUDE.md` files pointing back here.

## Conventions

Three modules, imported so they stay in context. The first two are portable and project-agnostic; the third holds the project-specific working rules.

@.claude/markdown-conventions.md

@.claude/git-conventions.md

@.claude/working-conventions.md
