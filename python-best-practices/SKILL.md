---
name: python-best-practices
description: Python best practices. Use when reading or editing any .py file.
---

# Python best practices

Apply the **type-system-discipline** and **separating-pure-from-io** principle skills first; this skill grounds them in Python syntax.

| Rule | Summary |
|------|---------|
| No dynamic attr access for known fields | `getattr`/`hasattr`/`setattr` on attributes known at design time defeats the type checker and hides typos. Use typed access. Model genuinely dynamic shapes with a dataclass, Protocol, TypedDict, or Pydantic model. |
| Typed records, not raw dicts | A dict passed around as a record loses its shape. Use a TypedDict, dataclass, or Pydantic model so fields and types are explicit. |
| No `Any`, annotate everything | `Any` disables checking wherever it touches. Annotate signatures. Treat external data as needing validation at the boundary, not `Any` inside. |
| Closed sets are Enums | Prefer a string enum (`class X(str, Enum)`) for a closed set, especially one referenced in more than one place or carrying behaviour. `Literal` is acceptable for a small local constraint on a single parameter. Never bare magic strings. |
| Named constants, no magic numbers | A literal number that carries meaning gets a named constant (module-level or `IntEnum`). The name is the documentation. |
| Catch specific, never swallow | No bare `except:` or broad `except Exception` that continues. Catch the specific type and handle it, or re-raise. A swallowed failure is a hidden bug. |
| Justify every `type: ignore` | Require `# type: ignore[code]` with the specific code and a one-line why. A bare ignore hides an unknown problem. |
| No mutable default args | `def f(x=[])` or `={}` shares state across calls. Default to `None`, build inside. |
| Dispatch, don't branch on type | Replace isinstance/elif ladders with polymorphism, a typed union plus `match`, or a dispatch table. |
| Compose over inherit | Favour composition (hold collaborators, depend on Protocols, inject dependencies) over inheritance. Use inheritance only for a genuine is-a with shared behaviour; reach for a Protocol when you just need an interface. |
| Pure core, thin shell | Keep computation out of I/O. Separate deterministic logic from DB, API, file, and log calls. See `separating-pure-from-io`. |
| Validate at the boundary, don't `cast` | `cast()` and unchecked assumptions about external data lie to the checker. Parse and validate where data crosses in, then trust the type. See `principle-boundary-discipline`. |
| Model variants, not Optional bags | A dataclass where every field is Optional to cover several cases hides illegal states. Model the variants so impossible combinations cannot be represented. See `principle-type-system-discipline`. |
| Explicit params, not `**kwargs` pass-through | `**kwargs` threaded through layers loses the signature and the types. Name the parameters. |
| In-body comment blocks signal extraction | Docstrings on module, class, and function are fine. A multiline comment narrating a step inside a function body usually means that chunk should be a named helper, or the logic should be simplified. Hide the complexity behind a good name. See `principle-minimize-reader-load`. |

Examples: `references/patterns.md`.
