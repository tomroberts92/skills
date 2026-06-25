# Working-style skills for Claude Code

A small, opinionated set of Claude Code skills for shipping software with discipline: plan before executing, deliver through small reviewed PRs, review against a real standards rubric rather than generic correctness, verify on a real deployment, and visualise systems as clean static pages.

Format-compatible with anything that reads a `SKILL.md` with frontmatter (Claude Code, and similar agents).

## Skills

| Skill | What it does | Invocation |
|---|---|---|
| [`tom-mode`](./tom-mode) | The working-style switch. Brainstorm and plan before code, run the right skill per phase, deliver through small reviewed PRs, verify on the real artifact. Bundles a shared **standards brief** and four **playbooks** (delivering a ticket, multi-subtask stack delivery, refactoring, responding to PR comments, verifying changes). | User-invoked (`/tom-mode`). Not model-auto-invoked. |
| [`tom-review`](./tom-review) | Reviews a PR or working diff against the standards brief plus the repo's own conventions. Owns standards, structure, and a diff-local correctness pass (null/empty/coercion/timezone/lifecycle/removed-guard); escalates to heavier reviewers rather than reimplementing them. Reports by default; `--comment` posts inline. | `/tom-review [PR]`, model-invocable. |
| [`html-explainer`](./html-explainer) | Builds a self-contained HTML explainer or slide deck in a warm-light editorial house style (single file, inline CSS, no build step). Explainer and deck modes, plus static / phase-machine / interactive-simulator archetypes. | `/html-explainer`, model-invocable. |
| [`python-best-practices`](./python-best-practices) | A smell table of Python rules (types, boundaries, enums over magic strings) with a `references/patterns.md`. Referenced by the standards brief. | Fires on `.py` files, model-invocable. |
| [`typescript-best-practices`](./typescript-best-practices) | The TypeScript equivalent, rules plus `references/patterns.md`. Referenced by the standards brief. | Fires on `.ts`/`.tsx`, model-invocable. |
| [`separating-pure-from-io`](./separating-pure-from-io) | Separate pure, deterministic logic from side effects (DB, API, files, logging) so the core tests with no mocks. Referenced by the standards brief and the refactoring playbook. | Model-invocable. |
| [`write-a-skill`](./write-a-skill) | Authoring guide for new skills: structure, progressive disclosure, bundled resources. Referenced by `tom-mode` for encoding repeat instructions. | Model-invocable. |

## How they fit together

```
/tom-mode  ──▶  brainstorm ──▶ write the plan ──▶ open a playbook ──▶ implement ──▶ verify ──▶ tom-review ──▶ PR
                                                   (delivering-a-ticket, etc.)        (verifying-changes)   (standards lens)
```

- **`tom-mode`** is the orchestrator. Its playbooks are the per-task skeletons; the **standards brief** (`tom-mode/references/standards-brief.md`) is the shared rubric pasted into every implementer and reviewer subagent, since subagents do not inherit the parent skill.
- **`tom-review`** reads that same brief, so what you build to and what you review against are one rubric.
- **`html-explainer`** is independent, for visualising a system or analysis.

## Dependencies

These skills are intentionally thin and **reference** other skills and tools rather than copying them. Install the ones you want the references to resolve. Nothing here breaks if a dependency is missing; the reference just won't fire.

**Referenced skills (install separately).** The bundled skills name these:

- **[pstack](https://github.com/cursor/plugins/tree/main/pstack)** (Cursor's skill collection) provides most of them: `architect`, `arena`, `how`, `why`, `interrogate`, `reflect`, `tdd`, `unslop`, and every `principle-*` the skills reference (`laziness-protocol`, `subtract-before-you-add`, `minimize-reader-load`, `foundational-thinking`, `type-system-discipline`, `boundary-discipline`, `migrate-callers-then-delete-legacy-apis`, `prove-it-works`). Install pstack and these all resolve.
- **[superpowers](https://github.com/obra/Superpowers)** plugin (the `superpowers:`-prefixed references): `brainstorming`, `writing-plans`, `subagent-driven-development`, `executing-plans`, `using-git-worktrees`, `test-driven-development`, `receiving-code-review`.
- **Other skills, by source** (tom-review escalations and craft references):
  - `thermo-nuclear-code-quality-review` — [cursor-team-kit](https://github.com/cursor/plugins/tree/main/cursor-team-kit).
  - `improve-codebase-architecture` — [mattpocock/skills](https://github.com/mattpocock/skills) (deep-vs-shallow modules).
  - `naming-things` — [kayvane1/skills](https://github.com/kayvane1/skills) (naming precision; not bundled here, since it is not mine to redistribute).
- **Claude Code built-in commands**: `verify`, `run`, `simplify`, `code-review`.

**Optional, repo-provided.** Used as examples, substitute your own:

- A ticket-kickoff skill (e.g. a `start-ticket`), a frontend-testing skill, observability conventions, and a stacked-PR helper (`gh-stack`). The playbooks degrade gracefully without them.

**External tooling referenced.** The `gh` CLI; your issue tracker's CLI; a browser-automation tool (for `html-explainer` and frontend verification); a database client or MCP and an observability-platform CLI (for `verifying-changes`).

## Install

Clone, then symlink the skills you want into your agent's skills directory:

```bash
git clone <this-repo> ~/code/skills

# Claude Code (user-level skills live in ~/.claude/skills)
ln -s ~/code/skills/tom-mode                  ~/.claude/skills/tom-mode
ln -s ~/code/skills/tom-review                ~/.claude/skills/tom-review
ln -s ~/code/skills/html-explainer            ~/.claude/skills/html-explainer
ln -s ~/code/skills/python-best-practices     ~/.claude/skills/python-best-practices
ln -s ~/code/skills/typescript-best-practices ~/.claude/skills/typescript-best-practices
ln -s ~/code/skills/separating-pure-from-io   ~/.claude/skills/separating-pure-from-io
ln -s ~/code/skills/write-a-skill             ~/.claude/skills/write-a-skill
```

`tom-mode` is set to user-invoked only (`disable-model-invocation: true`); trigger it with `/tom-mode`. The rest are model-invocable.

> **Name collision warning.** pstack also ships a `typescript-best-practices`. If you install both pstack and this bundle's version, two skills share that name. Pick one, this bundle's is its own rule set, not pstack's.

## Notes

- **Generalised from a private monorepo setup.** Tool names and repo specifics have been replaced with generic equivalents (preview/ephemeral environments, "your issue tracker", "your observability platform"). Where a playbook says "for example", swap in your stack.
- **`html-explainer/references/design-system.md`** is a warm-light editorial house style. Adjust the tokens to your own brand.
- **Attribution.** Referenced skills belong to their authors: `naming-things` is [kayvane1/skills](https://github.com/kayvane1/skills) (based on Ousterhout's *A Philosophy of Software Design*, Ch. 14); `improve-codebase-architecture` is [mattpocock/skills](https://github.com/mattpocock/skills) (same book's deep-module ideas); `thermo-nuclear-code-quality-review` is [cursor-team-kit](https://github.com/cursor/plugins/tree/main/cursor-team-kit); the `principle-*`, `architect`, `interrogate`, etc. are [pstack](https://github.com/cursor/plugins/tree/main/pstack); and the `superpowers:` skills are [obra/Superpowers](https://github.com/obra/Superpowers). None are bundled here.
- Add a `LICENSE` of your choice before publishing.
