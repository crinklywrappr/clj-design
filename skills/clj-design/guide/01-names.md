# Names

**Master criterion:** names should be **narrow** and **consistent**. Narrow = clearly excludes what it
cannot represent. Consistent = easily understood by someone familiar with the surrounding code, the
problem domain, and the Clojure ecosystem.

**Concept — sign / referent / sense.** A name's **sign** is its text; its **referent** is what it points
to; its **sense** is the fundamental properties we ascribe to it. (Frege's Morning/Evening Star: same
referent, different sense — *how* a name refers matters, not just what.) When we meet a new name we only
need its sense; the referent can change without us caring. A `var` is such a reference, so references
give code freedom to change over time — defining separate vars for one value signals we expect them to
diverge.

**Concept — a good name is indirection.** A name separates *what* from *how* (`get-jovian-moon` hides the
storage path `[:sol :jupiter]`); indirection is the foundation of software — layers peel away so we work
without understanding the whole. Names are the most common means of indirection.
> Abstraction vs indirection: 'abstraction' also names other concepts; 'indirection' is preferred
> because narrower (expanded in *Indirection*).

**Principle — sense should be timeless.** A name's sense should hold only timelessly-true properties;
treat implementation-dependent ones ("our ids are 128-bit") as effectively false, or you enshrine the
implementation and constrain future designs.

**Principle — narrow reveals sense; narrow ≠ specific.** Overly general obscures fundamental properties
and invites breaking changes; overly specific exposes the implementation and resists change; narrow
balances the two. Narrowness also comes from surrounding code, docs, and conversation — it can be
destroyed without touching code, and needs constant care.

**Principle — consistency minimizes confusion.** Sense often stays unspoken, so people ascribe different
properties and speak past each other; a name consistent with reader expectations costs less effort;
redefining a common name (e.g. `map`) forces readers to track context everywhere. Sense↔referent can also
be inconsistent (an `id` is "unique" but UUIDs only *probably* are) — an inconsistent name isn't
necessarily bad, since enforcing every invariant is impractical.

**Tradeoff (default+exceptions) — natural vs synthetic names.**
- **Natural** (`id`, `student`) — grasped immediately as one of many senses; novices participate now;
  everyone can reason by analogy (powerful when modeling the real world); but senses diverge and
  converge only slowly.
- **Synthetic** (`monad`, `tutelage`) — no intuitive sense, so unambiguous among experts; comprehension
  is binary (learn it or walk away); defies analogy and blocks novices.

Full consistency requires a 1:1 sign↔sense relationship, which pushes toward synthetic names; but a name
like `student` (a dozen senses) must be avoided or given separate contexts. Choose by audience: natural
for reach and analogy, synthetic for unambiguous expert communication.

## Naming Data

**Concept — what you control.** Every var, let-bound value, and parameter must be named. For an immutable
`var` you control sign and referent but not sense (people disagree what belongs in a set). For a
**parameter** you control only the sign — the value could be anything (types help but "type systems are a
tool, not a solution"). For a **let-bound** value you control the sign *and* the right-hand side.
Indirection hides: for vars the value; for parameters the caller's implementation; for let-bindings the
RHS expression.

**Directive — reuse the ecosystem's established names.** A can-be-any-map param is `m` (not `map`, which
shadows core); any key is `k`; an optional not-found arg is `not-found` defaulting to `nil`, matching
`get`.

**Tradeoff (default+exceptions) — data that "never changes."** Narrowing the name doesn't settle sense
disputes; you can freeze it with a comment, but if it truly never changes, consider whether it belongs in
a `var` at all — prefer a named constant (`Math/PI`) over a magic value (concision + fewer copy-paste
errors). Default: keep the `var` if it's used in many places; otherwise inline the value.

**Directive — enforce invariants at the top of a function**, not at every level (inter-function relations
aren't adversarial); put most invariant checks at the periphery, where the world *is* adversarial.

**Principle — Clojure's key property: syntax expresses semantic layers.** An experienced reader can skim
and still losslessly grasp intent — but only when side effects are avoided; skimmability relies on syntax
+ immutability. Self-evidency depends on the reader and erodes over time; deeper code has a smaller,
more expert audience, topmost names are read by everyone.

**Directive — prefer avoiding names.** Finding good names is hard, so avoid it: don't name every
intermediate — compose with `->>`/threading. If a function's implementation is clearer than any name,
make it anonymous (a large function isn't necessarily bad; if unwieldy but unnameable, leave it).

**Tradeoff (menu) — short name vs self-evident name** for a repeatedly-used value. A bare `g` means
nothing (forces reading both sides); a self-evident `surface-gravity` shadows the fn and is verbose. If
the left name isn't self-evident, make the right-hand expression as simple as possible — often by
extracting a self-evident intermediate:
```clojure
(let [surface-gravities (->> planets (remove gas-planet?) (map surface-gravity))]
  (doseq [g surface-gravities]...))
```

### Naming conventions (defaults from Clojure practice; prefer a codebase's own)

| Value | Name |
|---|---|
| can be anything (limit ops to `=`,`hash`,`str`) | `x` |
| a diverse range of datatypes (document them; prefer to `string-or-float-or-map`) | `x` |
| sequence of anything | `xs` |
| map any→any / arbitrary fn | `m` / `f` |
| sequences of maps / fns | `ms` / `fs` |
| self-reference (protocol/deftype/anon fn) | `this` |
| many same-type args | `[a b c... & rst]` (document the type) |
| arbitrary Clojure expression / macro body | `form` / `body` |
| map of K→V | `key—>value` (e.g. `class—>students`; nested `department—>class—>students`) |
| tuple of different types / its sequence | `a+b` (e.g. `tutor+student`) / `tutor+students` |

Prefer narrower names where possible (a well-defined map is `student`;
`students` a sequence — document shared properties).

**Tradeoff (menu) — compound datatype names.** Compounds are ambiguous (`tutor+students` = a sequence of
tuples or a tuple of students). If context won't disambiguate, invent a name (`tutelage`→`tutelages`) —
but that's synthetic, so document it and use only where readers have read the docs. Conventions exist to
defer inventing names until absolutely necessary.

## Naming Functions

**Concept — data scope.** The data visible from within your thread: parameters, let-bound values,
closed-over values, and global `var`s. A function can only **pull** new data into scope, **transform**
data already in scope, or **push** data into another scope. (Shared mutable state makes scopes
asymmetric: reading a public atom isn't a pull, but writing to it pushes local info to all threads.)

**Directive — one function, one action; compose the rest.** Most functions should only push, pull, OR
transform; at least one per process must do all three, but combined functions are hard to reuse — define
actions separately and compose.

**Directive — name boundary-crossers with a verb.** Pull → describe the datatype returned; push →
describe the effect; simultaneous push+pull → capture both and document. By effect: `get-payload`,
`delete-payload`, `compress-and-get-payload`.
> Outside Clojure's core, `get` should imply pulling data from another scope.
> Only trivial processes (like `echo`/`cat`) avoid doing all three actions (expanded in *Composition*).
>

**Directive — omit the datatype inside its namespace.** In a payload-specific namespace, use `get`,
`delete`, `compress-and-get` (callers use a prefix, `payload/get`). Shadowing core is safe/useful, but
declare it and define your version last so mid-file mistakes error rather than silently rebind:
```clojure
(ns application.data.payload
  (:refer-clojure :exclude [get]))
```

**Directive — verbs only when needed.** Pure transforms avoid verbs (`md5`, `timestamp`/`last-modified`,
`—>base64`; prefix in a broader namespace: `payload-md5`). Modifying data often needs a verb
(`add-student`); local ambiguity is resolved by the call's context. The `!` suffix for external effects
isn't universal — clarity comes from avoiding impure functions and documenting.
> The opening example should drop `get` and simply be `jovian-moon`.

**Principle — namespace cohesion.** A namespace should hold functions sharing a common purpose, so it
lends narrowness to their names (functions over a common datatype and/or data scope let you omit it).
Too many namespaces taxes readers — don't make one per table just to write `europa/get` vs `db/get-europa`;
add namespaces only when necessary, which implicitly questions new datatypes/scopes.

## Naming Macros

**Concept — two kinds: syntactic vs semantic.** `with-open` is understood syntactically; `core.async`'s
`go` (whose expansion even the authors can't describe; it skips anonymous fns) semantically.

**Principle — macros are poor indirection.** Syntactic macros require understanding their implementation
(they cut code volume, not conceptual burden); semantic macros require understanding the transformation's
semantics *and* failure modes. A good macro name signals it's a macro and prompts reading the impl; a
`with` prefix or a special-form name (`def`,`let`) should imply a predictable macroexpansion. Macros can't
be self-evident → the clarity of the expansion matters more than the name; macro names are usually
synthetic and need careful documentation.

**Tradeoff (default+exceptions) — large macros for a small audience.** Usually avoid; but a large macro
can be worthwhile to cut code size in low levels or tests — define and use it within one namespace, where
the name is unimportant as long as it isn't misleading.

**Narrowness and consistency are in tension.** Balance them by *audience* — which varies with the depth of
the code and shifts over time (success with an expert audience inevitably attracts less-expert readers).
Synthetic names have little downside once understood but are an obstacle for novices; natural names let
everyone make progress, at the risk of misunderstanding.
