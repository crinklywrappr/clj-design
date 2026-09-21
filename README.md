# clj-design

A [Claude Code](https://claude.com/claude-code) **skill** that applies the design philosophy of Zachary
Tellman's *Elements of Clojure* when you write, name, structure, or review Clojure code.

## What it does
- **Advise (default):** answers Clojure naming / idiom / design decisions using the book's choosing
  criteria and defaults, cited to the relevant chapter and lens.
- **Review:** walks a set of design checks over a file, snippet, or `git diff` and returns prioritized,
  *advisory* findings (not a dogmatic linter), each tied to a frame.
- **Smart loading:** a routing table pulls in only the relevant reference file per question, so it stays
  cheap until a Clojure design question actually needs it.

It carries the book's own stance: **these are defaults and heuristics, not laws — judged in context.**

## Install
Clone into your Claude Code skills directory:
```
git clone <this-repo> ~/.claude/skills/clj-design
```
Then, in a new session, it auto-triggers on Clojure design/naming/review intent, or invoke it with
`/clj-design`.

## Layout
- `SKILL.md` — the skill: trigger, stance, seven lenses, routing table, review procedure.
- `guide/` — the condensed reference (how-to + lenses, four chapters, code-time checks).

## Provenance & copyright — please read
The `guide/` content is a **condensed derivative of a copyrighted, commercial book**:

> *Elements of Clojure* © 2016–2019 Zachary Tellman. **All rights reserved.** — <https://elementsofclojure.com>

It was produced for **personal use** from a purchased copy. It is **not licensed for redistribution**, and
this repository should remain **private** unless and until the author grants permission. If you find this
useful, **buy the book** — it is excellent and far richer than any condensation.

All design insights are Tellman's; any errors introduced by condensation are not his. The skill scaffolding
(routing, review procedure, tooling) is the only original part.
