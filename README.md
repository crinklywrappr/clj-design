# clj-design

A [Claude Code](https://claude.com/claude-code) plugin for Clojure design. It applies the design
philosophy of Zachary Tellman's *Elements of Clojure* to how you name and structure Clojure code.

## What it does

- **Advise (the default):** it answers Clojure naming, idiom, and design questions with the book's
  criteria and defaults, and it cites the related chapter and lens.
- **Review:** it applies a set of design checks to a file, a snippet, or a `git diff`, and it gives
  ranked, advisory results. It is not a strict linter. Each result points to a frame.
- **Smart loading:** a routing table reads only the guide file for a question. Until then, the plugin
  stays cheap.

It keeps the book's own stance. These are defaults and heuristics, not laws, and you judge them in
context.

## Install

This is a Claude Code plugin. In Claude Code, add the marketplace, then install the plugin:

```
/plugin marketplace add crinklywrappr/clj-design
/plugin install clj-design@clj-design
```

In a new session it starts on its own for Clojure design, naming, or review work. You can also start it
with `/clj-design`.

## Layout

- `.claude-plugin/`: the plugin and marketplace manifests.
- `skills/clj-design/SKILL.md`: the skill file, with its stance, seven lenses, routing table, and review
  procedure.
- `skills/clj-design/guide/`: the condensed guide, with the how-to and lenses, four chapters, and
  code-time checks.

## Provenance and license

The content in `skills/clj-design/guide/` is a condensed, source-faithful distillation of Zachary
Tellman's *Elements of Clojure* (<https://elementsofclojure.com>), adapted with the author's permission.

This work uses the [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) license. See the `LICENSE`
file. If you give credit, you can share and adapt the work. The design insights are Tellman's. The
condensation, the seven lenses, the code-time checks, and the skill scaffolding are my own. If you find
this useful, buy the book. It is excellent, and far richer than this condensation.
