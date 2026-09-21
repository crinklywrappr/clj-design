# Elements of Clojure — How to Use This Guide

> This front section (lenses + throughlines) is a **reading aid synthesized from the book** — frames to
> think *with*. The four chapter files are **faithful to the text**. Where a throughline's cross-scale
> connection is the book's own, it's marked *(the book states this)*; where the unification is my framing,
> *(my framing)*.

## How to use this guide
These are **defaults and heuristics, not laws** — judged in context; where a codebase has its own
established practices, prefer them. The concepts here aren't a formula for correct design but **tools to
enumerate and discuss your options**. The aim is not to *solve* software design but to **reduce it to its
essential questions** — you supply the understanding of the software and its environment. Read the
**lenses** as the frames to think *with*, the **throughlines** as one idea recurring at different scales,
and the chapter files for the concrete rules.

## Core lenses — the seven frames to reason *with*

**L1 · Sense over sign.** A name has a **sign** (its text), a **referent** (what it points to), and a
**sense** (the fundamental, *timelessly-true* properties you ascribe). Readers only need the sense; the
referent can change beneath it. Make names — and interfaces — reveal a *narrow* sense: excluding what they
can't represent, exposing no implementation detail.

**L2 · Indirection = references (open) + conditionals (closed).** All indirection is built from two
primitives: a **reference** conveys values and is *open* (change behavior by supplying different values);
a **conditional** decides on values and is *closed* (change behavior only by editing code). Indirection
separates *what* from *how*, defines your code's layers, and tells the reader when to stop digging.

**L3 · A module = model + interface + environment.** Every abstraction is a **model** (data *and*
functions) exposed through an **interface** into an **environment** (everything else). The interface is
the *means of interaction* — not just functions, but effects and shared state too (`stdout`/`stdin`, logs,
shared atoms, network). Whatever the model omits is an **assumption** that the facet is fixed or
irrelevant; **invariants** enforce what it must never violate. To judge a module, first name its
environment.

**L4 · Software models are inductive — they satisfice, judged by their environment.** Unlike mathematics
(self-consistent, context-free) or physics (deductive, aspiring to predict), software reasons by analogy
and only needs to work *well enough* in its environment. Self-consistency ("correctness") is necessary but
not sufficient; usefulness is subjective and environmental.

**L5 · Pull / transform / push — keep the core pure, effects at the edges.** Computation moves data: a
**function** pulls into, transforms within, or pushes out of a *data scope*; a **process** does the same
across its *environment*. Put the **operational** work — pull/push: effects, I/O, invariant-enforcement,
and the *realization of lazy sequences* — at the **edges**, and keep the **functional** core (transform)
**referentially transparent**: no side effects, no reliance on dynamic scope, so it can be read, tested,
and reordered freely (this is what makes Clojure skimmable). Keep the edges thin and the core large.

**L6 · Three operations on data: accrete / reduce / reshape.** Every transform does exactly one:
**accrete** (add to a collection), **reduce** (discard information — treating different inputs as
equivalent), or **reshape** (move to a different collection, because the difference *does* matter). Name
which you're doing; keep accretion and reduction separable; never reshape implicitly. **Reduce is
abstraction itself, and carries the most value and the most risk.**

**L7 · Principled components, lasting interfaces.** Systems trade between **principled** (hierarchical,
interdependent, efficient, fragile) and **adaptable** (graph-like, redundant, flexible, robust). The
target is neither extreme: *principled components built to be discarded, separated by interfaces built to
last.*

## Throughlines — one idea operating at several scales

**T1 · Narrowness recurs at every scale.** Names should be narrow (reveal only their timeless sense);
interfaces should be kept narrow **"like names"**; models are useful because they're *small*. *(the book
links names↔interfaces; extending to models is my framing)*

**T2 · You assume what you ignore.** A name's unspoken sense, a module's omissions, and a process's
unhandled inputs are the same act — whatever you don't represent, you've assumed away. *(the book states
"by ignoring, we assume"; the cross-scale unification is my framing)*

**T3 · Abstraction = treating different things as equivalent.** Stated as a consequence in *Indirection*,
then reused as the very definition of **reduce** in *Composition*. *(the book states this — "all the
lessons of the previous chapter apply here")*

**T4 · Indirection defines layers and tells you when to stop reading.** A good name is indirection you can
trust without looking beneath; indirection "tells us when we're allowed to be incurious"; a process is
strong indirection you can understand one at a time. *(each instance is the book's; the throughline is my
framing)*

**T5 · Judge against the environment as it will be.** Usefulness depends on the environment's *future*,
not just its present: "over-engineered" means too few assumptions for what's coming; systems are judged
against a constantly-changing environment. *(book-stated at each point; the unification is my framing)*

**T6 · Effects and laziness live at the edges; the core stays referentially transparent.** The same
discipline drives the state idioms (don't let dynamic scope or lazy realization escape pure code — the
`binding`, atom/STM, and lazy-pull warnings) and the process phases (pull *invokes* transform; operational
vs functional namespaces). This is the most code-shaping Clojure discipline in the book. *(a synthesis of
book-stated instances)*
