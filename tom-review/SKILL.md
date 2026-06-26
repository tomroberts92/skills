---
name: tom-review
description: "Review a PR or the current diff against Tom's standards (the tom-mode standards brief), composing thermo-nuclear-code-quality-review and interrogate. Use for /tom-review, reviewing a PR against my standards, or looping review over open PRs. Default reports findings; pass --comment to post inline PR comments."
disable-model-invocation: false
---

# Tom review

Review a PR or the working diff against Tom's standards. This is the "review against my standards" lens, distinct from generic correctness review. It composes the heavy reviewers rather than reimplementing them.

## Scope

- A PR by number or URL (`/tom-review 1694`), or the current working-tree diff if none is given.
- Fetch the diff with `gh pr diff <n>` for a PR, or `git diff main...HEAD` for the current branch.

## Getting the code

Most of the review needs only the diff; the codebase-context and repo-convention checks need the tree. Get them in this order, and do not default to a full clone.

1. **Diff and metadata.** No clone needed. `gh pr diff <n> --repo <owner/repo>` and `gh pr view <n> --repo <owner/repo> --json title,body,files,additions,deletions`. The diff-local checks run off this alone.
2. **Context and conventions, prefer a local checkout.** If the repo is already checked out locally (e.g. a local monorepo checkout), read it read-only with `git -C <path>` and absolute paths, and never switch its branch. For a file's exact PR-branch state use `git -C <path> show <ref>:<path>` or `gh api repos/<owner/repo>/contents/<path>?ref=<branch>`, not a checkout.
3. **No local checkout, fetch what you need.** Pull the convention files and any suspected-duplicate files via `gh api repos/<owner/repo>/contents/<path>?ref=<branch>`, and use `gh search code` for the reuse checks. That covers conventions and most reuse questions without cloning.
4. **Last resort, shallow clone.** Only if a deep reuse audit needs the full tree and the repo is not local: `git clone --depth 1 --filter=blob:none` into a temp dir, grep there, then remove it. A large monorepo clone is the expensive path; avoid it unless steps 1 to 3 fall short.

## Review against the standards brief

Read `~/.claude/skills/tom-mode/references/standards-brief.md` and check every change against it: the laziness and YAGNI ladder and reuse, the code rules (`python-best-practices` / `typescript-best-practices`, `separating-pure-from-io`, the type, boundary, and reader-load principles), the naming rules (`naming-things`), the abstraction and failure-mode rules (deep modules over leaky pass-throughs, fail loudly with no silent fallbacks, specified exception handling), the comment rules, the test bar, and the verification bar. For `.py` and `.ts`/`.tsx` files lean on the matching best-practices skill directly.

## Apply the repo's own conventions too

The brief is generic. Every repo has conventions it does not encode, and the repo's own reviewer enforces them, so load and apply them as well or you will miss exactly the findings the repo's bot catches. Look for the repo's `CLAUDE.md` (and per-app `CLAUDE.md`), its `.claude/rules/*`, and the domain skills it ships under `.claude/skills/`. For example, a repo may ship its own observability conventions (metric naming, an operation-tag registry kept current in `CLAUDE.md`, log-level rules) in a skill or rules file; load and apply those alongside the brief. Check changes against the repo's conventions, not just the generic ones here.

## Check the diff-local correctness family

Beyond the standards, own the bugs visible in the diff and its immediate context. For each changed function or component, walk its inputs against the edges from the brief rather than just confirming it reads cleanly: null and undefined, empty collections, type coercion (`String(null)` is the truthy string `"null"`, `==` vs `===`), boundary and zero values, timezone and locale (`toISOString` is UTC, not the user's calendar day), async and lifecycle staleness (a value frozen at mount, a stale closure), and any guard the diff removed. These are statically spottable, and they are the family a standards-only read skims past, the empty-chart, open-to-404, and timezone-drop class.

This lens stops at what the diff shows. A deep adversarial pass on a risky change is `interrogate`'s (four models, main thread); behaviour that only appears when the code runs is the `verifying-changes` loop's; the git-history and prior-PR hunt, or an auto-posted filtered comment, is the heavy `/code-review`'s. Flag those if you see them, but they are not this lens's guarantee.

## Escalate, do not duplicate

- **Structure, not optional on a large diff.** Run `thermo-nuclear-code-quality-review` over the diff whenever it adds a new module, touches five or more files, or changes roughly 1000 or more lines. Do not judge whether it "needs" it; a large diff is exactly where cross-cutting structure, file growth, duplication, and silent-drop or observability asymmetries get missed. A smaller, contained diff may skip it.
- **Leaky abstractions and shallow modules.** When the diff adds a shallow pass-through, or an interface that leaks its implementation, lean on `improve-codebase-architecture` for targeted deepening suggestions (the deletion test, deep versus shallow, seams). Offer them as feedback; a large deepening is usually a follow-up, not a blocker on this PR.
- **Deep adversarial pass, when the change is high-risk or contested.** Run `interrogate`: four reviewers on four different models tear the diff apart, with agreement across models as the high-confidence signal. This is the go-to escalation when the diff-local pass is not enough, and it is well-suited to the blind-spot edges (null, empty-state, timezone) a single reviewer aims past. Run it from the main thread, not a delegated review subagent, since a subagent cannot spawn the multi-model fan-out; if a finding inside a subagent needs it, surface it for the parent.
- **Runtime correctness.** Behaviour that only appears when the code runs is the `verifying-changes` loop's, not a static review's.
- **The heavy `/code-review` plugin, only for its niche.** Reach for it when you specifically want the git-history and prior-PR-comment hunt, or the auto-posted, false-positive-filtered PR comment. For a plain "tear this apart" pass, `interrogate` is lighter and already covers it, so do not run the 15-agent plugin by default. A repo's own automated PR review is a complementary pass on top, not a substitute for the lenses above.

## Delegating to a subagent

A fresh subagent can run this review (report-only is the usual mode). It reads the brief and the thermonuclear rubric directly from `~/.claude/skills/`, which requires read access to that path (allowed at the user level). It applies the thermonuclear rubric by reading its SKILL.md rather than invoking it, and interrogate stays with the parent. If a repo's permissions deny a subagent read access to `~/.claude/skills/`, the parent inlines the brief into the hand-off prompt so the review stays faithful instead of falling back to whatever the subagent had in context.

## Output

Default: report findings back, grouped by severity, each with the file, the rule it breaks, and the fix. Do not post to the PR. The severity buckets and rule labels are for this internal triage view only.

With `--comment`: post the findings as inline PR comments, like `/code-review --comment`. Write them per `~/.claude/skills/tom-mode/references/pr-comment-voice.md` (your voice, the comment not the conversation, lead with the point and mark severity once). On top of that shared note, two rules are specific to this skill, because only tom-review has a rubric sitting behind the comment:

- **Drop the graded ladder.** The severity buckets and rubric-category tags belong to the internal report, never the posted text. Strip "Should-fix:", "Blocking (verification bar):", "(type discipline)", and the redundant "(non-blocking)" after a nit. Severity rides on the verdict (request-changes vs comment), not a bracket grading every line.
- **Do not name the rubric.** Say what is wrong in plain terms, never "the kind of thing the standards flag" or "the verification bar".

## Verdict and approval

End with a verdict, not just a list. The verdict covers this skill's lens: the standards brief, the repo's conventions, structure, and the diff-local correctness family above. It is not a full correctness sign-off, runtime correctness is the `verifying-changes` loop's and a deep adversarial pass on a risky change is `interrogate`'s. A clean tom-review means clean on that lens, not bug-free everywhere, so say so when you approve, and it is not mistaken for the last word before merge.

- **Approve** when nothing blocks on that lens: no structural regression, no standards or convention violation, and no diff-local correctness bug that has to change before merge. Optional or minor suggestions do not block; leave them as non-blocking comments.
- **Request changes** when something blocks: a structural regression, a standards or convention violation, or a diff-local correctness bug that must change. For a correctness issue that needs running the code or deep context, flag it and route it to verification or the bug-hunt, but do not treat its absence as something this lens verified.
- Otherwise just **comment** (questions, non-blocking suggestions) and leave the call to the author.

Judge the substance. The repo's branch protection already gates merge on green checks and unresolved comments, and resolving threads is the author's job, so do not withhold approval over thread housekeeping. The only question is whether the findings block.

## Looping over PRs

Runnable under `/loop` to babysit a PR toward approval. Each pass re-reviews after a push or on an interval, checks whether the previous pass's findings have been actioned, and drives the PR to a verdict.

- Re-review only what changed since the last pass; do not repost unchanged findings.
- Each pass, report what is still blocking, plus `reviewDecision` and checks (`gh pr view <n> --json reviewDecision,statusCheckRollup`), so the loop surfaces progress. That is information; branch protection enforces the actual merge requirements.
- When the prior findings are actioned and nothing blocks, notify that the PR is ready. By default the loop stops there for you to give the approval. Pass `--approve` to let it post the approval itself (`gh pr review --approve`) on a clean verdict, never while anything blocks; a still-blocking finding stays a request-changes.
