# Git conventions

How commits are written and history is shaped in this repository.

*Portable module — no project-specific content. Drop it into any repository's `.claude/` and reference it from `CLAUDE.md` with `@.claude/git-conventions.md`.*

## Commit messages

**Absolutely follow the [cbea.ms seven rules](https://cbea.ms/git-commit/) of a great commit message:**

1. Separate subject from body with a blank line.
2. Limit the subject line to 50 characters.
3. Capitalize the subject line.
4. Do not end the subject line with a period.
5. Use the imperative mood in the subject line ("Add", not "Added"/"Adds").
6. Wrap the body at 72 characters.
7. Use the body to explain **what and why**, not how.

These are **hard limits, not targets.** Rules 2 and 6 are the only two that can be checked mechanically, and they are the two most often missed, because a 51-character subject looks exactly like a 49-character one. Measure before committing rather than judging by eye:

```
git log --format='%s' <range> | python3 -c "import sys
[print(len(l), l) for l in sys.stdin.read().splitlines() if len(l) > 50]"

git log --format='%b' <range> | python3 -c "import sys
[print(len(l), l) for l in sys.stdin.read().splitlines() if len(l) > 72]"
```

Both must print nothing. **Do not use `awk 'length($0)>50'` for this** — most `awk` implementations count *bytes*, so a line containing an em-dash, a curly quote, or any non-ASCII character reads as longer than it is and reports a violation that is not there. A check that cries wolf gets ignored, which is worse than no check. The limits are in characters, so count characters. Run them over the range about to be pushed. "Close enough" is how a limit erodes, and the correction afterwards costs a history rewrite — and a force push, if the commits already left the machine. Rule 1 deserves the same suspicion: a heredoc that drops the blank line after the subject silently produces a commit whose entire body is folded into its title.

Rule 6 is not in tension with the no-hard-wrap Markdown convention — a commit message is not Markdown and is not rendered. Wrap commit bodies at 72; leave `.md` prose unwrapped. Issue and task titles use the same imperative mood as subject lines.

## Atomic commits

- Each commit is one logical change and stands on its own.

- Sequence commits so the history tells the story of the work.

- Commit often. Stage and show the diff before committing, so the author can follow at reading speed.

## Attribution

- **Commit messages keep a "Claude" co-authorship credit but no email address.** Strip the `<noreply@anthropic.com>` part, because Git platforms would otherwise turn it into a clickable link to a dead mailbox. Accept the trade knowingly: without an address the trailer is no longer machine-parsed as co-authorship, so the credit reads as plain text instead of registering in the contributors UI.

- **Never put Claude-Session or `claude.ai` URLs in a commit message.**

## History

- **Never wrap a single commit in a merge bubble.** A one-commit branch is rebased onto `main` and fast-forwarded, so history stays linear. `--no-ff` exists to record that several commits belong together; around one commit it records nothing the commit does not already say, and costs a node and two lines of graph to say it. Reserve it for a genuinely multi-commit line of work.

- **Fold small corrections into an unpushed tip with `--amend`** — check `origin` first to confirm it is unpushed.

## Working copies

- **Default to `main` or an in-repo branch.** Ask before creating a worktree; agents never create their own.
- **Never use bare `git stash` / `git stash pop`.** The stash stack is shared across worktrees.
