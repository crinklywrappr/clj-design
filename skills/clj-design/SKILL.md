---
name: clj-design
description: Elements-of-Clojure design guidance for writing, naming, structuring, and reviewing Clojure. Use for Clojure naming decisions; choosing idioms (atom vs STM, option maps, binding/laziness, nil, do, letfn); module/abstraction/dispatch design; process & pipeline structure; and design reviews of Clojure code or diffs.
---

# clj-design — the Elements of Clojure design lens

A condensed, source-faithful distillation of Zachary Tellman's *Elements of Clojure*, plus a review
procedure. **Stance: these are defaults and heuristics, not laws — advisory, judged in context. Prefer a
codebase's own established conventions. Present this as one opinionated school, not the only way; the book
is explicit that software design has no closed-form solution.**

## The frames — reason *with* these

- **L1 Sense over sign** — a name should reveal a *narrow*, timelessly-true **sense** (what it is), not
  its *how*; the same goes for interfaces.
- **L2 Indirection = references (open) + conditionals (closed)** — it separates *what* from *how*, defines
  the code's layers, and tells the reader when to stop digging.
- **L3 Module = model + interface + environment** — whatever the model omits is an *assumption*;
  invariants enforce what it must never violate; the interface is the boundary of interaction (functions
  **and** effects/shared state), not just functions.
- **L4 Software models are inductive** — they *satisfice*, judged by their environment; self-consistent
  ("correct") is necessary but not sufficient for *useful*.
- **L5 Pull / transform / push** — keep the functional **core referentially transparent**; put effects,
  I/O, and lazy-seq realization at the **edges**.
- **L6 Accrete / reduce / reshape** — the only three data operations; **reduce is abstraction** and
  carries the most value and the most risk.
- **L7 Principled components, lasting interfaces** — principled parts built to be discarded, separated by
  interfaces built to last.

## How to load (progressive — read only what the task needs)

Reference files are bundled **alongside this SKILL.md, in the `guide/` directory** (resolve them relative
to this skill's own location). **Always start** with `guide/00-how-to-use-and-lenses.md` (the full frames + throughlines), 
then read by topic:

| The question is about… | Read |
|---|---|
| naming, vars, params, namespaces, macros | `guide/01-names.md` |
| state (atom/ref/STM), `binding`/laziness, `nil`, `do`, `letfn`, option maps, interop, `for`, accessors | `guide/02-idioms.md` |
| abstraction, modules, dispatch (protocol/multimethod), interfaces, assumptions, principled vs adaptable | `guide/03-indirection.md` |
| processes, effects, pipelines, pull/transform/push, accrete/reduce/reshape, tasks/protocols/topology | `guide/04-composition.md` |
| **any design review** | also `guide/05-checks.md` (the checks) |

Load one or two files, not the whole guide. When a question straddles topics, load the closest chapter
plus `00`.

## Modes

### Advise / consult (default)
For a naming, idiom, or design decision: load the frames + the matching chapter, then answer using the
book's **choosing criteria and stated defaults**. Give the default *and* the condition to deviate. Cite
the source inline, e.g. "(Names / L1)" or "(Idioms — atom vs STM)". Note where local context might
override the default.

### Review
When asked to review Clojure (a file, a snippet, or `git diff`): load `00`, `05-checks.md`, and the
chapters matching the code's concerns; walk the checks and lenses over the code; report findings as:

> **`<file:line or form>`** — `<the issue in plain terms>` — *lens/rule + source* (e.g. "Names / L1");
> **suggested change:** `<concrete fix>`. *(when this might be fine: …)*

- Order findings most-consequential first; be specific; cite the guide.
- Keep it **advisory, not a linter**: flag, explain the reasoning, suggest — don't demand.
- Respect existing conventions. If the code is well-designed, say so plainly.

## Don't
- Don't fire on trivial or purely stylistic points the book doesn't address.
- Don't present these as hard rules.
- Don't rewrite code wholesale under the banner of "design" — target the specific design issue.
- Don't drown the answer in citations; cite the frame that matters and move on.
