# Responding to PR comments

For "address the review comments", "respond to the PR feedback", "handle the review on PR N". You own the response to every thread.

1. **Gather every open thread first.** `gh pr view <n> --comments` (and `gh api` for review threads if needed). Read them all before acting. Do not respond thread-by-thread in isolation; a later comment often reframes an earlier one.
2. **Triage each comment.** Classify it: a valid change, a question to answer, a disagreement, or out of scope. Use `superpowers:receiving-code-review` for the judgment, not performative agreement. Weigh the concreteness of the argument, not the reviewer's seniority. A specific failure-mode argument outweighs a generic best-practice appeal. No "great catch!" reflex.
3. **Make the changes per Code discipline.** A behaviour change gets a failing test first; keep the suite green. Group related fixes.
4. **Reply to every thread explicitly.** State what you changed and where, or, if you disagree, the reason, candidly. For out-of-scope items, link a follow-up ticket rather than expanding the PR. Write replies clean, per the global writing rules.
5. **Prove it on the real artifact** (tests green; `verify` / `run` if behaviour changed). Own the verification, do not trust a "looks good".
6. **Commit small and ordered, then push** (tom-mode: Process). No ticket refs in code or comments.
7. **Re-trigger review.** Pushing does not always re-run an automated reviewer. Refresh it per the repo's mechanism (some setups need `gh pr ready --undo && gh pr ready`, or an `@claude` comment). Re-request human review when the changes are substantive.

**Done means:** every thread is changed-and-replied, answered, pushed back on with a reason, or ticketed; tests green; review re-requested.
