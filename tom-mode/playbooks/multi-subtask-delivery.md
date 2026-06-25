# Multi-subtask delivery

For a ticket too large for one PR. Split it into dependent subtasks and deliver each as its own PR in a stack. This playbook runs `delivering-a-ticket` once per layer in a single shared worktree, and adds only what is stack-specific: the `gh stack` mechanics, the cascading restack, and a whole-stack review. Read `delivering-a-ticket` for the per-ticket loop. Get sign-off on the split and on each layer's plan before writing code.

Subtasks in a stack are dependent (each builds on the one below), so work the whole stack in a **single feature worktree** with `gh stack`, not one worktree per subtask. A single dir is what lets `gh stack` navigate and cascade-rebase the chain when a lower layer changes, and it keeps every layer's branch and review in one place. See the `gh-stack` skill for the commands and git-config prerequisites.

The standards brief is the shared rubric at `../references/standards-brief.md`, used by this playbook, `delivering-a-ticket`, and the `tom-review` skill. Paste it into every implementer and reviewer subagent prompt.

## Set up the stack

1. **Brainstorm and split.** If the ticket is large or under-shaped, brainstorm first and split into dependent sub-tasks ordered bottom to top, each shippable as its own PR. Create the sub-task tickets in your tracker (use its ticket-creator skill if it has one).

2. **One feature worktree, named for the feature.** Create a single worktree for the whole stack off main, named `stack-<feature>` (a short slug for the stacked effort, with the `stack-` prefix so it stays distinct from the normal per-ticket `<ticket>-<desc>` worktrees). Do not name it after the first subtask ticket, and do not create a worktree per subtask; it holds every layer. Use `superpowers:using-git-worktrees` with that name (or a `start-ticket` skill, overriding its ticket-derived worktree name to the feature slug). The per-layer branches created with `gh stack` keep their per-ticket `<subN>-<desc>` names; only the worktree directory is feature-named. Gather the feature context once (the parent epic and the subtask tickets) so it carries to planning every layer. Run the session rooted at this worktree so `subagent-driven-development` can write to it; if you run from the main checkout instead, probe write access to the worktree before dispatching, since a subagent's writable root is the session cwd.

   **Already have the branches in separate worktrees?** For a stack that predates this flow (branches developed one-worktree-per-subtask), gh stack cannot rebase them while they are checked out in worktrees. Clear the per-branch worktrees first (confirm each is clean and pushed), then adopt the branches into one clone with `gh stack init <bottom> ... <top>` and manage the stack from there. See the `gh-stack` skill.

## Per layer, bottom to top

3. **Open the layer with `gh stack`.** `gh stack init <sub1>-<desc>` for the bottom layer, then `gh stack add <subN>-<desc>` for each layer above it. This creates and checks out the branch in the feature worktree, no new worktree. Pass full branch names (no prefix). This replaces the worktree kickoff (step 1) of `delivering-a-ticket`. See the `gh-stack` skill.

4. **Deliver the layer with `delivering-a-ticket`.** Follow that playbook from its planning step onward: plan the subtask (reusing the feature context already gathered), implement with `superpowers:subagent-driven-development` and the per-task two-stage review and the standards brief, verify against the real artifact with representative data, watch the net diff, and run a per-layer `tom-review` before the PR. Save the layer's `tom-review` summary to `.local/reviews/` in the feature worktree (gitignored, alongside `.local/plans/`) so the whole-stack review can build on it. The layer's PR stacks on the layer below.

Then move to the next layer (back to step 3, `gh stack add`) until the stack is complete.

## Finalize the stack

5. **Create the stacked PRs.** `gh stack submit` (or `gh stack link <bottom> ... <top>` if the branches were managed outside gh stack). Each PR's base is the layer below it. The GitHub Stack object needs the repo to have stacks enabled (the beta); without it you still get correctly chained dependent PRs, just not the Stack view. If the repo does not have stacks enabled, `submit` prints `Stacked PRs are not enabled for this repository` and exits RC=0, a warning not a failure, with the chained PRs intact. Do not halt on that warning.

6. **Restack when a lower layer changes.** When review feedback lands on a lower layer (handle it per `playbooks/responding-to-pr-comments.md`), navigate to it (`gh stack down` or `gh stack checkout`), make and commit the change there, then `gh stack rebase --upstack` to cascade it up the stack and `gh stack sync` to push and update the PRs. This cascading rebase is the reason for the single-worktree model.

7. **Final whole-stack `tom-review`.** When the stack is complete, spawn a fresh subagent to run `tom-review` across the whole stack (`git diff main...<top-of-stack>`) in report mode, scoped to cross-subtask emergent structure no single-layer review could see, such as an abstraction duplicated across layers, a file grown past a healthy size across several of them, or concepts that should have been unified. At this scope tom-review's size triggers fire the thermonuclear structural rubric. Do not re-review each layer's internals; the per-layer passes did that. Hand it the per-layer review summaries from `.local/reviews/` (all in this one worktree) so it builds on them rather than re-deriving the stack. Timing matters. If lower layers merged as they went these findings are mostly follow-up tickets; if the stack is still open they can still drive a restructuring before merge.

**Done means:** each subtask shipped as its own reviewed PR via `delivering-a-ticket`, the stack created and kept current with `gh stack`, the completed stack passed a final cross-subtask `tom-review`, net complexity not silently growing.
