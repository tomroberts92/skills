# Refactoring

You own the contract. The structure changes; the behaviour does not. For "refactor", "rename", "extract", "inline", "dedupe", "restructure", "move this module", "tidy up this area". If the cleanup reveals a missing feature or a real bug, split it out and ship the structural change first against the pinned contract.

Get sign-off on the plan before writing code (see tom-mode: Plan before executing). The steps below are the plan's skeleton; `superpowers:writing-plans` fills them in for the specific refactor.

1. **Pin the behaviour contract.** Run `how` over the affected subsystem to learn the contract, then write a characterization test (or snapshot / equivalence harness) capturing current behaviour before any structure moves (`principle-prove-it-works`). No coverage in the area? Write the pin first. Type-check and lint are not a pin.
2. **Name the target shape.** State what the module layout, types, and call graph should be if built today (`principle-foundational-thinking`). If the target crosses a function boundary, run `architect` to explore the shape before moving.
3. **Subtract before adding.** Delete dead weight, collapse one-caller wrappers, drop redundant validators before introducing the new shape (`principle-subtract-before-you-add`, `principle-laziness-protocol`). The smallest change that reaches the target ships.
4. **Move in small behaviour-preserving steps, each keeping the pin green.** For API reshapes, migrate every caller and delete the old API in the same wave (`principle-migrate-callers-then-delete-legacy-apis`) — no shims, no parallel old-and-new paths. Spot-check every rename against the actual files; renames silently miss usages in strings and back-references. Delegate the mechanical edits to a code-writing subagent (sonnet-class) with a specific scope (file paths, the names being moved, the behaviour to hold); review the diff yourself.
5. **Prove behaviour is unchanged on the real artifact**, not "it compiles" (`principle-prove-it-works`). Keep the characterization test green; for larger reshapes run an equivalence check (diff old-vs-new outputs, or a smoke run via `verify` / `run`). Own the verification — don't trust a delegate's "looks good".
6. **Confirm the change earns its place.** Success is reduced reader load (`principle-minimize-reader-load`): fewer layers between question and answer, less hidden state, fewer indirections without a second consumer. If the diff doesn't lower reader load somewhere, revert it.
7. **Commit and open a PR (tom-mode: Process).** Small ordered commits that tell the story — subtraction, then the reshape, then follow-on cleanup, so one revert undoes one slice. Branch or worktree, never main; small focused PR; link the tracking ticket in the branch and PR; wait for review, no self-merge.

**Done means:** structure changed, the pin held, behaviour proven unchanged on the real artifact, reader load lower, no new behaviour. Tests green, PR approved and merged.
