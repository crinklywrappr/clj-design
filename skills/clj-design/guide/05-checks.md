# Checks to Run While Writing

> Interpretation layer — the book's principles restated as questions to run at code-time. The chapter
> files hold the source guidance these derive from.

## Naming (L1)
- Is this name **narrow** — does its sense consist only of *timelessly-true* properties, and exclude what
  it can't represent?
- Would a reader guess the function's **purpose** from its name without reading the impl? If it describes
  *how* not *what*, rename.
- Am I about to **shadow a core name** (`map`,`get`,`key`)? If so, is it `:refer-clojure :exclude`'d and
  defined at the **bottom of the namespace** (so a mid-file mistaken use errors instead of silently
  rebinding)?
- Is this a **natural name with many senses** readers could diverge on (like `student`)? Give it a
  narrower context (a dedicated namespace) or use a synthetic name + docs.

## Purity & effects (L5, T6) — the highest-frequency checks
- Is my **functional core** free of side effects and dynamic scope — **referentially transparent**?
- Am I returning a **lazy seq** that could be realized *outside* the scope that makes it valid
  (`binding`/`with-db`/transaction)?
- Does the **pull phase *invoke* the transform**, rather than returning an effectful lazy-seq that forces
  the *caller* to make the operational decisions (retry / truncate / let it throw)?
- Are **effects flagged** so a reader notices them — an explicit `do`, a `_`-binding, or *negative space*
  (each effect on its own line in a `let` body) — not hidden inside a form's implicit `do`?
- Are **operational and functional** concerns in **separate namespaces**?

## Data & transform (L6)
- For this transform: am I **accreting, reducing, or reshaping** — and did I make it obvious?
- Is this a **reduction** in the L6 sense — *discarding information* (a sum, a lookup, a dedup), not
  necessarily Clojure's `reduce`? That's the high-value / high-risk step; give it extra care.
- Am I **reshaping implicitly**? Demand the shape the function needs (if it needs a set, take a set).
- Am I passing an **ambiguous `nil`** downstream? Resolve it (default / keyword sentinel) or throw.

## Modules & abstraction (L3, L4)
- Have I **described this module's environment** before calling it useful?
- **What am I ignoring** here — is that assumption sound for the environment *as it will be*?
- Is this abstraction merely **self-consistent ("correct")**, or actually **useful** in its environment?
- Can this model **fit in the head of someone other than me**? If not, shrink it or reconsider.
- Am I enforcing a function's invariants **at its top** (trusted internal callers) and validating input
  **at the system's periphery** (the adversarial outside) — not re-checking at every level?

## Indirection & systems (L2, L7)
- Where **competing extensions or definitions are possible**, am I using a *closed* conditional rather
  than an *open* table?
- Is this interface **single-purpose**? Then it tends to drift (both sides grow interdependent) — don't
  over-invest in maintaining it; only genuinely many-directional interfaces are worth stabilizing. And
  don't expose a **foundational** interface to the outside world before it has matured internally.
- Am I building **principled components separated by durable interfaces**, grouping modules by shared
  assumptions?

## A default the book leaves implicit
- **Method dispatch — [my synthesis, not the book's]:** default to **protocols** for type-based
  polymorphism (keeping either the protocol or the class a hidden impl detail); reach for **multimethods**
  only when dispatch must depend on more than the first argument's type; avoid Clojure **hierarchies**.
  The book gives the tradeoffs and cautions but states no default — this ordering is my interpretation.

## Lens map — which frame governs which material

| Lens | Governs |
|---|---|
| L1 Sense over sign | all of *Names*; interface narrowness (*Indirection*) |
| L2 References/conditionals | dispatch, tables, open/closed (*Indirection*) |
| L3 Model+interface+environment | the module framework; assumptions/invariants/conventions |
| L4 Inductive models satisfice | useful-vs-correct; judging models |
| L5 Pull/transform/push, pure core | data scope (*Names*); `binding`/atom/`do` idioms; process phases |
| L6 Accrete/reduce/reshape | the transform phase; `nil` handling |
| L7 Principled components, lasting interfaces | *Systems of Modules* |
