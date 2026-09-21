# Composition

**Concept — composition.** Combining separate abstractions into a new one; once combined, they define each
other's environment and create the context by which each piece is judged. "Composition is applied
abstraction." Its ultimate goal is not new functions but **processes**.

**Concept — a process = pull / transform / push.** A **process** has execution isolation (when it runs),
data isolation (where it runs), and is sequential (an OS process, a thread, an async-callback chain…). It
is the smallest unit of standalone computation, and provides strong indirection from its environment, so
we can understand a system one process at a time. Software that doesn't pull, transform, AND push at least
once is only useful combined with other software.
> Exception: some math (pi digits, PRNGs, fractals) generates data from no input — but it's only useful
> fed into other processes.

**Principle — you can only tell yourself what you already know.** Pulled data may be eventually available,
but parameters must be immediately available; in isolation a process's parameters are limited to what's at
hand, so if we're the only source, software only returns variations on what we already know.

## A Unit of Computation

**Concept — data isolation.** A process sees only global data or its parameters; once initialized its
universe is bounded (immutable values + mutable references). Inter-process communication is only via
shared references; any change to one is an **effect**, often behind an interface. Even a thread-safe
structure isn't enough — you must *signal* an update to trigger another process; hence queues
(thread-safety + signaling).
> "Effect", not "side effect" — the latter implies effects are incidental/avoidable, but they're a
> necessary component of software.

**Directive — keep mutability at the edges.** Passing a mutable structure into a function risks a new edge
(it may share it with another process, now or later) → implicit process composition makes the system
unreasonable. Use internal mutable structures only in principled components.

**Concept — execution isolation.** Processes run sequentially; function invocation has an immediate,
ordered effect. Inter-process communication is different — you can update+signal a shared reference but
can't control when/how another process reacts; by delaying effects, a process forces you to wait. Reason
about a process in isolation when active; when paused, consider the surrounding processes.

**Tradeoff (menu) — bound how long a process waits**, so it stays considerable in isolation.
- **Low expectations** — be content with current performance (hardware exceeds most software's demands);
  reduces how often you must hold the whole system in your head.
- **Timeouts** — cap the wait; on timeout, don't diagnose, recover.

These combine, and only work where performance isn't primary and some failure is acceptable; in stringent
domains, instead fight to keep the whole system small enough to hold in your head.

**Concept — execution model.** A process's strategies for when its environment provides too much or too
little (ignore, retry, backoff, fail) are its **execution model**; a well-defined one lets the process be
considered in isolation. Queues alone don't provide isolation — they couple processes and let one block
another indefinitely; without wait-limits, coupled processes share one execution model. Sometimes this is
unavoidable (file-system components sit beneath many apps and can only wait), but an execution model
spanning too many processes exceeds understanding.

*(Concept — bidirectional: "pull" and "push" both usually need two-way communication — signal readiness /
confirm downstream capacity. Frameworks like Ring invert control: they invoke your code, freeing you from
effect complexity but making production behavior harder to understand.)*

## Building a Process

**Directive — keep the phases separate until the last moment.** Pull, transform, and push should be
composed only at the process's apex (a parameterized `(defn repl [read eval print]...)` — each phase
understandable in isolation). Push/pull are **operational** (code in motion; define the process's limits;
enforce context-specific invariants); transform is **functional** (code at rest; defines purpose; ignores
much context).

**Principle — leaky processes are always dangerous.** Clojure's `sort` works only if conventions are
followed (`(sort (range 1e12))` OOMs); using it without enforcing its assumptions creates a leaky
abstraction. *Tradeoff:* leaky abstractions are fine *within a principled component that shares their
assumptions* (after `slurp`ing a config, `sort` is harmless — already fragile to oversized inputs). But a
principled component can't span processes (process boundaries are strong indirection) — at process edges,
always enforce the assumptions within.

**Directive — the pull phase does more than acquire.** It also verifies shape/size and defines what
happens when data is invalid or unavailable. A bare lazy-seq over file lines ignores failure modes
(unbounded line → OOM on `next`; malformed JSON/EDN; `IOException`), making them fatal. In a robust
process, the **pull phase should *invoke* the transform phase** (flexible error response); if it just
yields a lazy-seq, control flow inverts and constrains you. *Tradeoff:* failing is sometimes fine — config
files are special (reviewed by the same people; bounded; read at deploy with people on hand) — but most
input isn't bounded or monitored, so most processes must be intrinsically robust.
> Alternatively, make systems robust to *process loss* (Erlang OTP) — but that needs language/runtime
> support absent in Clojure, so it's out of scope here.

**Framework — transforming: only three things you can do with data.**
- **Accrete** — add to a collection (when you don't know enough yet, or want a larger batch).
- **Reduce** — discard information (inputs yielding the same output become interchangeable). This *is*
  abstraction — treating different values as equivalent; it carries the **most value and most risk** (why
  analysts distrust mean/variance), so it needs the most care.
- **Reshape** — move to a different collection to make accretion/reduction easier (a DB over flat files
  for random access). *Not* abstraction — motivated by differences that *do* matter. (Data
  structures/algorithms is the study of these shape tradeoffs — often overstated, but knowing Clojure's
  core-structure tradeoffs usually suffices; not ignorable.)

**Directive — never implicitly reshape.** If a function needs a set, demand a set (lets others judge fit;
prevents repeated reshaping). Keep accretions and reductions separable and expose them as composable
pieces.

**Concept — pushing: a descriptor of effects.** The transform phase outputs not just data but a
**descriptor** of effects to perform; the push phase acts on it. Simplest case, the descriptor is the data
(`"hello world!"`→`println`); complex effects correspond directly and may also specify *how*:
```clojure
{:url "http://example.com" :method :post :body "hello world!"
 :follow-redirects? true :max-redirects 99}
```
Don't assume a descriptor has well-defined semantics — data has no intrinsic meaning; its semantics are
the effects it produces when passed into functions.
> A descriptor may also describe a *pull* (an HTTP GET); the design considerations are largely the same.
>

**Principle — delay committing data to a function.** The push phase begins wherever functions perform
effects — i.e. wherever the meaning of data is defined. You can't introspect a closed-over effectful
function (`printer`) — only invoke it, and only alter what comes before/after (`prepend`/`append`). A
function can only be accreted, never reduced or reshaped; exchanging data for a function gains semantics
but loses almost everything else, so delay it as long as possible.
> Some effects can be redirected via dynamic state (`println` by binding `*out*`), but that's only needed
> when data was discarded prematurely — prefer altering the descriptor.

**Principle — operational edges, functional core.** A process = operational phases at the edges + a
functional phase in the middle. Operational phases guard against production pathologies (the only true
test is to deploy); done well, they let you understand and test the functional phase in isolation
(Clojure's REPL strength). Keep most code in the middle, little at the edges — and keep operational and
functional phases in **separate namespaces**, letting functional namespaces reference only other
functional namespaces, to stop operational concerns creeping inward.

## Composing Processes

**Concept — topology.** For many apps one process doesn't suffice; compose them into a system whose graph
(active processes + relationships) is its **topology**. A process is mostly opaque and is *not* a value —
refer to it via a process identifier or a channel.

**Concept — resolution & routing.** In a static topology, channels suffice (a bash pipeline: anonymous,
externally-configured stdin/stdout) — but only if the system never changes. Otherwise identifiers connect
to processes via **resolution** (DNS: name→IP, one-to-many for load-balancing/failover); little
distinguishes resolution from **discovery**. A **router** exposes one channel and distributes its data
across processes (even a thread pool is a router).

**Concept — tasks & protocols.** Most data pushed to another process is a **command** describing an
effect, passed along a chain until it can be realized. A push's proximate goal is communication; its
ultimate goal is completing a **task** — which begins when a command enters the system and ends where an
effect's consequences become uncertain (you can write to a DB but not dictate it's read).

**Directive — acknowledge and remember tasks.** Because processes/channels are unreliable, acknowledge
completion back up the chain; remember incomplete tasks in a **single place at the system's edge** so
other processes can forget a command once passed along. The formal mechanism to accomplish + acknowledge
tasks is a **protocol** (TCP/HTTP across one edge; system-level protocols across the whole topology).

**Closing.** A task ends where an effect's consequences become uncertain — but that's where a system's
value begins; systems aren't intrinsically useful, they're judged against a constantly-changing
environment. This framework can't tell you if your software is useful; it's a structure into which you
contribute your own understanding. *"We cannot solve software design, but we can reduce it to its
essential questions."*
