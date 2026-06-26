---
name: tom-mode
description: "Tom's working style: brainstorm and plan before executing, show reasoning, TDD with covering tests, small reviewed PRs linked to tickets, and reuse over reinvention. Use for tom, /tom-mode, or requests to work in Tom's style."
disable-model-invocation: true
---

# Tom mode

The user's working conventions. Follow them unless the user overrides in the moment.

## Plan before executing

Get sign-off on the approach before doing the work, including on reversible changes. Substantial work follows the superpowers skills in order:

1. **Brainstorm.** Large or creative work starts with `superpowers:brainstorming` to explore intent and requirements before any solution.
2. **Write the plan.** Turn the shaped work into a detailed step-by-step implementation plan with `superpowers:writing-plans`.
3. **Execute the plan.** Two routes. If the user wants subagents to run it in this session, or says "subagent-driven", use `superpowers:subagent-driven-development` (fresh subagent per task with a two-stage review after each task, run before the next task starts, never batched at the end; inject the standards brief from `references/standards-brief.md`). If the user will run it themselves in a fresh session, use `superpowers:executing-plans` (single session model, checkpoints), and add a `Recommended execution model:` line to the plan header. When the plan is written and the route is unstated, ask which.

If the repo provides a ticket-kickoff skill (e.g. a `start-ticket` skill), start there. It creates the worktree with the repo's own conventions and routes to brainstorm-or-plan, so prefer it over invoking the worktree and planning skills directly.

For a trivial one-off edit, skip the ceremony, but still say what you're about to do first. "Just do it" autonomy is not the default; show the path before you walk it.

## Playbooks

When a task type has a bundled playbook below, open it and copy its steps into your todolist before writing any task-specific plan. The playbook is the skeleton; `superpowers:writing-plans` fills it in for the specific task. A step you skip stays in the list with a one-line `skip: <reason>`.

- **Refactoring.** A behaviour-preserving change to structure (rename, extract, inline, dedupe, move). `playbooks/refactoring.md`.
- **Responding to PR comments.** Addressing review feedback on an open PR. `playbooks/responding-to-pr-comments.md`.
- **Delivering a ticket.** Implementing one ticket end to end: plan, subagent-driven implement with the per-task two-stage review, verify, `tom-review`, PR. `playbooks/delivering-a-ticket.md`.
- **Verifying changes.** Proving a change end to end on a real deployment, beyond unit tests: seed the dimension that exercises it, deploy to an ephemeral env, drive real data, then inspect the data store, the runtime signals, and the UI. `playbooks/verifying-changes.md`.
- **Multi-subtask delivery.** A ticket too large for one PR, split into a stack of PRs. Runs `delivering-a-ticket` per layer in one `stack-<feature>` worktree, plus the `gh stack` mechanics and a whole-stack review. `playbooks/multi-subtask-delivery.md`.

Feature and bug-fix work routes through `superpowers:brainstorming` → `writing-plans` and, in repos that provide one, a ticket-kickoff skill like `start-ticket`. No bundled playbook needed yet.

## Skills by phase

- **Brainstorm / scope.** `how` for code walkthroughs and placement questions. `why` for design rationale and regression history.
- **Design.** `architect` for any change that crosses a function boundary. `interrogate` for a contested decision before committing. `arena` when a single attempt would lock in the wrong shape.
- **Write the plan.** Encode the skill invocations above as explicit plan steps. Execution subagents do not inherit this skill, so the plan is the only carrier across that boundary.
- **Reflect.** `reflect` after a substantial session to route learnings back into skills.

## Writing the reply

Write the reply clean as you draft it. The cleanup-afterward pass fails, so don't generate the bad sentence in the first place. Apply the `unslop` skill to every line — it must always apply, and it already bans the AI tells (em dashes, colons as mid-sentence connectors, inline-header lists that restate the line).

- **Show your reasoning.** Explain the thinking as you go, not just the result. Name what you chose and what you chose against. A bare conclusion with no path behind it is a miss. Keep it operational, not a lecture.
- **Terse is not an excuse to drop content.** Be concise, but keep the details, tradeoffs, choices, and open decisions the task calls for.
- **Frame impact for the consumer and the maintainer.** Name who the work is for (an end user, a colleague importing the library) and what changes for them, before any implementation detail. Then what the next engineer who owns this code inherits. If you can't say what either would notice, the work or the explanation is off.
- **Never fabricate a link, citation, or reference.** Link only artifacts you produced or read this session.
- **The artifact is not the conversation.** Anything you post or commit on the user's behalf (a PR comment, a review reply, a commit message, a PR body) carries the content, never the working chat that produced it. No "as we discussed", no instruction you were handed. For PR comments specifically, see `references/pr-comment-voice.md`.

## Verification

A change is not done when the code compiles. It is done when:

- The tests pass.
- The PR is approved and merged.

For a bug, reproduce it first, then fix it, then add a covering test that fails without the fix. No repro means no confirmed fix. Reach for the `tdd` skill (or `superpowers:test-driven-development`) on bugfix and feature work. Use the `verify` and `run` skills to confirm behavior against the real artifact when tests alone do not prove it.

When the change has runtime behaviour that unit tests do not exercise (a data path, a pipeline stage, an API a UI consumes), the `playbooks/verifying-changes.md` playbook is the full loop: seed the dimension that exercises the change, deploy to a real environment, drive real data through, then inspect the data store, the logs and traces, and the UI. This step is still manual prompting; the playbook is the checklist that makes it repeatable.

## Code discipline

- **TDD.** Write the failing test first, then make it pass, then refactor.
- **Pure core, thin shell.** Keep business logic in pure functions; concentrate I/O and validation at the boundaries so the core tests with zero mocks. See `separating-pure-from-io`.
- **Simplify as you go.** Run `/simplify` on a diff that has grown layered, before review.

Lean on these principle skills — read the leaf skill in full when one applies, don't restate them:

- `principle-subtract-before-you-add` — an addition or refactor: remove dead weight first, build on the simpler base.
- `principle-laziness-protocol` — tempted to add abstractions or layers: bias to deletion and the smallest change that solves it.
- `principle-minimize-reader-load` — code that's hard to trace: collapse one-caller wrappers, shrink mutable scope, count the layers.
- `principle-foundational-thinking` — before writing logic: name the core types and data shapes first.
- `principle-type-system-discipline` — designing types or a signature: make illegal states unrepresentable, parse external data at the boundary.
- `principle-boundary-discipline` — wiring validation, errors, adapters: guards at boundaries, trust internal types, keep logic pure.
- `principle-migrate-callers-then-delete-legacy-apis` — new internal API while old callers exist: migrate and delete in one wave, no shims.
- `principle-prove-it-works` — before declaring done: verify against the real artifact, not "it compiles".

## Comments

Readable code is the goal. Comment where the code is complex or what it does is non-obvious, and for a non-obvious *why* the code itself can't show. Skip the comment where the code already says it. Write comments clean as you go. Do not add a noise comment and remove it later. This applies to every file you produce, including a delegate's diff and any verify or test script.

- Docstrings on modules, classes, and functions are fine. A multiline comment block narrating a chunk of logic inside a function body is a smell. It usually means that chunk should become a named helper, or the logic should be simplified. Hide the complexity behind a good name rather than annotating it in place. See `principle-minimize-reader-load`.
- No phase-narration in scripts. Delete `// Phase 1: add cards`; the assertion or log string is the only doc you need. Write `assert(ok, 'persisted across restart')`, not a `// move the card` comment above the code.
- Don't restate what the code already says, and don't duplicate the commit message's rationale inline.
- Never put ticket numbers or references in code comments. Ticket linkage belongs in the branch and PR.

## Process

- Work on a branch or a git worktree. Never commit directly to the default branch.
- Open a PR and wait for review. No self-merge of unreviewed work.
- Keep PRs small and focused. Prefer several narrow PRs over one fat one, and stack follow-ups.
- Link the work to its tracking ticket through the branch and PR. Your issue tracker's CLI and the `gh` CLI are available.

## Subagents

Delegate only bulk or heavy work: large reads, mechanical sweeps, wide fan-out. Keep judgment, synthesis, and the final decision in the main thread. Do not spawn subagents for work you can do inline without filling the context window.

## Skills and tooling

- **Reuse over reinvention.** Reach for the repo's existing design system, component library, and UI-development skill before building from scratch. Let the active repo's own context and skills name the stack and components, do not assume one. Frontends differ across repos (one on shadcn/ui + Tailwind, another on Vuetify, another on Bootstrap), so the repo's own guidance picks the stack, not this skill.
- **Fix broken skills in their own PR.** If a skill is wrong mid-task, fix it separately. Do not silently work around it.
- **No hardcoded project values.** Skills and scripts take variables with sensible defaults, never project-specific values baked in.
- **Encode repeat instructions.** When the same correction comes up a second time, propose making it a skill, a lint, or a check rather than repeating it. Use `write-a-skill` to author one.
