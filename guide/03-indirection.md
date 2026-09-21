# Indirection

**Concept — indirection.** Separation between *what* and *how*; it exists wherever "how does this work?"
is best answered "it depends." Useful when the implementation is complex or changing; it defines
software's layers and tells the reader when to stop exploring.

**Concept — its two primitives: references and conditionals.** A **reference** points to a referent
(a name is dereferenced at compile time, a pointer at runtime; a reference can point to nothing — a
compile error, or `nil`). A **conditional** (`if`/`case`) makes behavior depend on values. A reference
conveys values; a conditional decides on them — complementary, composing into arbitrary complexity.

**Principle — references are open, conditionals are closed.** You change a reference's behavior by
conveying different values; you change a conditional only by changing code. You can't fake an "open
conditional" with a data structure of predicates+clauses — conditional code is **ordered** (overlapping
predicates make order matter) and that order lives in the populating code, so it stays closed.

**Concept — a table.** An open decision mechanism must be **unordered**: a structure with a distinct set
of keys. To stay useful it must avoid conflicts — keep it private, or extend it only with private keys.
Where conflicts are possible, use conditionals (they make an explicit, closed decision).

## Method Dispatch

**Tradeoff (menu) — interfaces vs protocols vs multimethods** (all table-based; efficiency is inversely
proportional to openness).
- **Interfaces** — dispatch on class, impl must live in the class → conflicts impossible, static dispatch
  possible.
- **Protocols** — dispatch on class but anyone may relate protocol↔class → no static dispatch, may need
  reading the whole codebase, public+public risks conflicts. *Keep either the class or protocol a hidden
  impl detail.*
- **Multimethods** — dispatch on a key from all args → most flexible, some perf cost; avoid public
  multimethod↔key relationships (redefining `print-method` for `PersistentVector` would be disastrous).

Hierarchies (overlaps resolved by `prefer-method`) add closed, override-anywhere decisions — rarely worth
it. Protocols/multimethods rely on shared global state (incremental dispatch without threading a table) —
usually worthwhile.

**Escape-hatch — explicit dispatch table.** When that tradeoff fails, define `class—>method—>impl` + an
`invoke` helper and pass the table as a parameter — ungainly but lets you change dispatch locally. Rarely
needed, always within reach.
> Can't replace the explicit table param with dynamic scope — incompatible with lazy evaluation.

## What is an Abstraction?

**Concept — Hoare's definition + invariants.** A structure has a **concrete representation** (internal
model) and an **abstract representation** (exposed interface); the **abstraction function** maps
concrete→abstract. An **invariant** is a property every operation maintains ("list of unique integers"):
given the invariant, a method's correctness needs only the model, and the model can change (e.g. become
sorted) without changing interface semantics.
> Clojure favors 32-element blocks (vectors, chunked-seqs) because rising reference cost made cons cells
> impractical; the meaningful exception is syntax trees (small, rarely processed at runtime).

**Principle — self-consistency is necessary but not sufficient.** Mathematical "correctness" =
self-consistency, which by omission deems context unimportant (an early marine clock's needs weren't
invariants but **assumptions** it couldn't enforce). In software, models must be self-consistent AND
useful in their environment; self-consistency is objective, utility subjective — so distrust easy
answers and be curious.

## A Model for Modules

**Core framework — a module = model + interface + environment.** The **model** is data + functions; the
**interface** is how model and environment interact; the **environment** is everything else (components,
users, the world). A model reflects chosen facets to narrow attention — everything it omits is an
assumption those facets are fixed or irrelevant; if it can represent invalid states it must enforce
invariants.

**Concept — software models are inductive.** Deductive reasoning maps environment→model and infers
necessary conclusions (physics aspires to this but falls short). Software reasons by analogy: if two
things share a model-point, observing one predicts the other. Inductive conclusions are contingent but
more resilient; they **satisfice** (work well enough) rather than predict. To judge any model, first
define its environment.

**Principle — easy to make a deductive model, hard to make one *useful*.** (Brooks's "nine women can't
make a baby in one month" refutes a model that is simple, deductive, and *wrong*.) Distrust any
real-world abstraction that touts its own logical simplicity; deductive models don't belong at the
periphery.

**Concept — invariants, assumptions, conventions, interfaces.**
- **Invariants** — needed when a model can represent invalid values; also about model↔environment fit
  (email regex only narrows to *superficially* valid — real validation must reach outside the process).
  Model↔environment drift is silent; periodic revalidation only *mitigates*.
- **Assumptions** — everything omitted; small models carry enormous numbers of them, most sound. Reduce
  fragile ones by layering modules with complementary invariants — *"abstractions that fail together
  should stay together"* (a mutex around a non-thread-safe collection; validate input once at the outer
  layer).
- **Conventions** — an unhidden assumption becomes yours to enforce (leaky abstractions: C++'s manual
  `free` → RAII; ch2's idioms are conventions for what Clojure drops on us). A useful tool, not a
  solution.
- **Interfaces** — how model and environment interact (formal + effects/shared state); they describe a
  module's **sense**, so they change slower than models and almost always grow. Keep them narrow, like
  names.
- **Environments** — an undifferentiated "everything else"; to judge usefulness you must draw a line
  (domain-expert, case-by-case) around the facets worth attention. That underdefined quality is what lets
  us import other fields' insights (all fields have model/interface/environment).
> A facet only reflects reality if we interpret its meaning carefully — a `can-fly?` field reflects a
> user's willingness to *claim* they can, not the fact.

## Consequences of our Model *(aphoristic principles)*

- **To abstract is to treat different things as equivalent** — ignoring parts of the world lets us apply
  knowledge to novel situations; including a facet means a change to it invalidates past understanding.
  Induction = deciding which differences to acknowledge (include a facet only if plausible changes would
  alter the model's identity).
- **Models reflect our perception of their environment** — no objective measure of a facet's importance;
  by curating what a model reflects we can distort the environment unrecognizably.
- **A module is useful only if its assumptions are sound** — by ignoring, we assume (bare-string SQL
  assumes no malicious input). Judge assumptions by what the environment is AND will become;
  "over-engineered" = too few assumptions, a property of intended use, not of the software. State the
  environment (present + future) up front.
- **To know a module's assumptions, know its model** — they're duals. Software we can't understand
  becomes **oracular** (trusted/distrusted blindly; ML generates oracles) — it makes us perpetual
  novices.
- **A module cannot prevent its own misuse** — more abstraction only trades failure modes; know a
  module's assumptions and anticipate trouble → use it only when it's useful. Understand the
  *consequences* of an implementation, not just the implementation.
- **If a model ignores too much: grow it, replace it, or narrow its use** — add the facet, rebuild, or
  "fire your customer" ("never meant to be used that way").
- **Models are useful because they're small** — growth reduces assumptions (more robust) but harms
  understanding; manage the *rate* of growth for yourself, but to teach it, absolute size matters — if
  only an expert can hold it in their head, consider discarding it.
- **Starting from scratch is costly** — reuse over-engineered legacy layers; their generality adds
  robustness to change. When replacing software, cut only what you can no longer use.
- **If a module makes unrealistic assumptions, users can wrap it, build conventions, or discard it** —
  fragile assumptions are OK because complementary alternatives exist; if you don't solve a user's
  problem, someone else will.
- **If a module cannot be discarded, it may destroy what it doesn't reflect** — mandatory abstractions
  are coercive (they force the environment to fit them; enterprise software chosen for us). Creators
  can't ignore their models' impact.
- **Software would be easy if things never changed** — it must change with its environment or become
  useless; little software keeps value without continuous change → stop comparing software to civil
  engineering. Minimize how changing one component affects others.

## Systems of Modules

**Tradeoff (menu) — principled vs adaptable systems.** Changing code risks invalidating assumptions
elsewhere; two strategies minimize this:
- **Principled** (hierarchy, minimal internal indirection) — top-down principles let components assume
  more → smaller, faster, elegant, learnable top-down; but highly interdependent (a house of cards) and
  fragile (built to a fixed purpose, can't be reoriented).
- **Adaptable** (graph, high internal indirection) — components blind to neighbors' internals →
  redundant, larger, less efficient, but flexible; resists incremental decomposition (no root/leaves →
  exhaustive exploration).

When two principled systems share a component, an interface is born; its permanence enables surrounding
change. A single-purpose interface tends to shift (both sides grow interdependent, it becomes vestigial);
an interface pulled in many directions is intrinsically stable.

**Thesis — the target.** Don't build a wholly principled system: build **a collection of principled
components, built to be discarded, separated by interfaces built to last** (a *complex adaptive system* —
the ecosystem, not the organism, adapts). Clojure is mostly unprincipled; its few principles
(immutability) promote this.

**Directive — interface longevity.** An interface disappears only when every participant on one side
does; foundational interfaces (TCP/POSIX) outlast application ones (REST). Avoid creating foundational
interfaces — they calcify when exposed to the world; if you must, let one mature within your own code
first. Modules with common assumptions share a component; dissimilar ones stay separate → small,
predictable change.

**Principle — bias check.** A stable, uniform domain needs little indirection — but any poorly-understood
domain *seems* stable, and the industry is biased toward simple solutions born of incuriosity. Clojure's
opposing adaptable bias is reasonable, but the right balance needs deep understanding of the software,
its environment, and what both may become.
