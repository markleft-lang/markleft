# Markdown conventions

How every `.md` file in this repository is formatted. Anything not covered here follows standard [markdownlint](https://github.com/DavidAnson/markdownlint) rules.

*Portable module — no project-specific content. Drop it into any repository's `.claude/` and reference it from `CLAUDE.md` with `@.claude/markdown-conventions.md`.*

## The conventions

- **No hard wrap.** Body prose is one line per paragraph — don't wrap at 80 or any other column. Reflowing a wrapped paragraph rewrites every line and buries the real edit in end-of-line churn, where an unwrapped paragraph diffs at the word level. Let the renderer soft-wrap (equivalent to `pandoc --wrap=none`). This matters most wherever documents are revised by amendment rather than rewritten, because there a clean word-level diff *is* the audit trail.

- **Blank lines between list items — per list, not per item.** If a list has long, multi-sentence items, separate every item with a blank line; if its items are short, keep the whole list tight. Decide once for the list and never mix the two within one list.

- **Blank lines around block elements.** Every heading, list, code fence, and table gets a blank line above and below (markdownlint MD022 / MD031 / MD032).

- **Header comment banners are the exception.** If a file grows an `<!-- … -->` context or attribution block at the top, hand-wrap it to ~80 columns. Those are fixed blocks that never get reflowed, so wrapping costs no diff churn.

## Scope

These conventions govern how Markdown *source* is formatted, and they apply to every `.md` file in the repository — including `CLAUDE.md` and anything under `.claude/`. They say nothing about which Markdown constructs a project permits; that is a separate question, and a project that restricts constructs states so in its own `CLAUDE.md`.

**Markdown does not wrap; commit bodies do.** If the repository also follows the companion git conventions, note that commit message bodies wrap at 72 columns. A commit message is not Markdown and is not rendered. Do not let either convention "fix" the other.

## Existing files

Files already hard-wrapped may stay that way until they are next edited, then get unwrapped as part of that edit. Reflowing a whole tree in one pass is also fine, but do it as its own commit so the reflow never hides a content change.
