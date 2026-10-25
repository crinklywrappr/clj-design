# Idioms

Idioms map code structure to intent; used consistently, they let
readers trust their intuition. Every meaningful surprise costs reader confidence — software that
constantly surprises is poorly written.

**Prefer `<` and `<=`.** — Order inequalities least-to-greatest. `>` points at nothing, forcing the
reader to mentally reorder terms; a consistent direction removes that burden (`<` = upward slope, so
`(< a b c)` ascends). The choice of `(< a b)` vs `(> b a)` is usually arbitrary — pick one. *Except:* in
a series of near-identical predicates over the *same terms*, keep the terms in one order and let the
operator vary (even using `>`) rather than reorder terms to stay least-to-greatest:
```clojure
(cond (< a b) …  (= a b) …  (> a b) …); clearer — terms fixed, operator varies
(cond (< a b) …  (= a b) …  (< b a) …); strict `<`, but the flipped (b a) reads worse
```

> *Related:* variadic `-`/`/` are easy to misread — `(- a b c…)` ≡ `(- a (+ b c…))` and
> `(/ a b c…)` ≡ `(/ a (* b c…))`; usually best to write the associativity out explicitly.

**If a function accumulates values, support every arity** (0/1/2 + variadic). — `(reduce f coll)` throws
on empty unless given an init; core fns supply their own via 0-arity. Only 0- and 2-arity are
interesting (1-arity returns its arg, variadic reduces); pick a reasonable 0-arity default (it isn't
always obvious — `conj` returns a vector). This is the **monoid** shape (0-arity identity + 2-arity
combine of *same-type* values — which is why `conj`, combining a collection with elements, is *not* a
monoid); `reduce` still permits accumulators over dissimilar types like `conj`.
```clojure
(defn concat
...
  ([a b & rst] (reduce concat (concat a b) rst)))
```
*Except:* skip full arities if the fn is rarely used / unlikely to grow, or has no sensible 0-arity
default — then give only 2-arity (callers must supply reduce's init).

**Use an option map, not named parameters.** — For optional values take a single map as a positional
param; not `&`-named params, not multi-arity positional defaults.
```clojure
(defn pi [n {:keys [efficiently? correctly?]
             :or {efficiently? true correctly? true} :as options}]
...)
```
*Why:* multi-arity positional optionals force an importance ordering (specifying a later param forces the
earlier ones); `& {:keys...}` named params let you skip/change/add without touching call sites but each
call builds+looks-up a hash-map AND they don't compose (passing options deeper needs `(apply concat)`).
A plain map param avoids both — it passes as-is to inner functions; the only cost is an extra pair of
braces. The natural shape of options is a map; named params are harmless only when always hand-written
(in practice, only macros). *Tradeoff (perf):* both add measurable overhead — in perf-sensitive contexts
use positional params only.

**No one should have to know you've used `binding`.** — *(Motivation: threading a new optional parameter
like `turbo-mode?` through a whole call chain `a→b→c` is invasive — every intermediate function grows the
param, and multi-arity defaults tax every caller; that pain is what tempts us toward a dynamic var.)*
Don't expose dynamic scope across a public API: a public `binding`-based macro (`slowly`) is a poor design
choice — callers can't see it and will forget. Confine `binding` to a function whose result depends only
on its parameters:
```clojure
(defn a
  ([x] (b x))
  ([x turbo-mode?] (binding [*turbo-mode?* turbo-mode?] (b x)))); c won't run lazily
```
*Why:* dynamic scope (and side effects) break **referential transparency** — an expression should be
interchangeable with its result and return the same value regardless of where it's evaluated. Laziness
relies on it, and `binding` breaks it: a lazy seq returned from a `binding` scope is realized *outside*
it (chunked-seqs realize 32 at a time → the bug hides until data grows / only in prod). An internal
`binding` can only be misused by editors of the callee; an exposed macro by anyone. *Tradeoff:* dynamic
scope is still powerful for connecting code without editing everything between, and for testability (a
`binding` at a test's top level over a defaulted dynamic var is safe) — but any dynamic var invites
re-binding; `with-redefs` may be safer.

**If you have mutable state, use an `atom`.** — *(Decision: in-process mutable state — atom vs STM.)*
- **Single atom holding an immutable map** — simplest, lowest overhead, faster under low/no contention;
  one container hits the ~60% utilization retry threshold sooner.
- **STM (refs in `dosync`)** — retries only when *your* refs are touched (better under real contention),
  but breaks referential transparency (lazy `alter`/`commute`/`ref-set` in the wrong transaction;
  `commute`'s return may differ from the committed value) and makes consistent snapshots costly
  (`ensure`, not `deref`).

Default (per book): **one atom.** Ladder if it isn't enough: spread work across more processes → split
into smaller independent atoms → only then STM (write-heavy loads not offloadable to a database). Agents
(unbounded queue) are best avoided.

**An explicit `do` block implies side effects.** — Ignoring a return value is a side effect; a
`nil`-returning fn exists solely for side effects. Many forms have an implicit `do` (`fn`,`let`,`loop`,
`when`) and don't invite close reading, so flag side effects there. *Tradeoff (menu):* techniques — a
redundant `do`; negative space (statements on their own lines in a `let` body); or a `_`-binding for an
inline effect. Any is fine, if used consistently.

**Use the narrowest possible data accessor.** — `(keys m)` over `(map key x)`/`(map first x)`; prefer
`get`/`nth`/`contains?` over invoking a collection as a function. *Why:* one structure wears many guises
(vector-as-map, map-as-entry-seq, array-as-lazy-seq), so there are many equivalent paths that differ for
the reader — `(map first x)` says "seq of seqs", `(map key x)` "seq of entries", only `(keys x)` "map".
Clearer names help but the clarity must be mirrored in the code; generic accessors imply the datatype
differences don't matter.

**Use `letfn` only for mutual recursion.** — Elsewhere prefer `let` with `(fn name [...])`. *Why:*
`letfn`'s binding form is structurally irregular (both sides in one form) and makes even experienced
readers stumble; its only real value is out-of-order references between functions.

**Java interop should be obvious.** — Avoid macros like `..` that disguise interop as ordinary Clojure;
unusual code should look unusual. *Why:* Clojure data is understood via its innards (nested
maps/sets/vectors/seqs); a Java object via its name + docs — a different reader mindset. `..` hides `put`
as if a Clojure fn, erasing that distinction.

**Use `for` for cartesian products** — unadorned by `:let`/`:when`/`:while`.
```clojure
(for [a [1 2 3] b [:a :b :c]] [a b]); every combination
```
*Why:* transformations are best piecemeal via `->>` (steps easily reordered); `for` covers simple
map/filter but not `group-by`, so a comprehension sits awkwardly in a larger chain — reserve it for its
unique ability (cartesian product). It also suits declarative data literals (hiccup templates), but avoid
special clauses there too.

**`nil` should be the absence of only a few values.** — Interpret/resolve `nil` at regular intervals
rather than passing an ambiguous `nil` downstream. *Concept:* `nil`'s meaning is context-dependent —
absence of a seq (`conj`/`cons`/`nth`), empty coll (`count`), absence of a map (`assoc`/`get`), of truth
(`if`), or of whatever you looked up; Clojure treats any non-map as an empty map. *Why:* you can't judge
a `nil`-returning fn in isolation — downstream may read it differently; composing nil-friendly fns
explodes ambiguity, and an NPE forces walking back through hypotheses. Resolve with explicit defaults, and
prefer a keyword sentinel (harder to ignore than number-or-nil):
```clojure
(-> key->numbers :c (conj 8 9 10));;=> (10 9 8); nil read as a seq
(-> key->numbers (:c []) (conj 8 9 10));;=> [8 9 10]; explicit absent value
(-> solar-system :jupiter :callisto (:mass :mass-not-found))
```
Avoid `(defn f [x & args] (when x...))` — it passes the buck; coerce `nil` to an empty collection if you
can, else throw.
