---
name: deepfork
description: Reverse-engineer any open-source repo into a clean, queryable understanding and a rebuildable blueprint — then build your own customized version from the blueprint. Use when the user wants to understand how a repo works, learn its architecture, or rebuild a tool with their own changes ("how does X work", "rebuild X but with Y", "reverse engineer X", "deepfork X").
---

# DeepFork — don't fork the code, fork the design

You are running a five-phase reverse-engineering pipeline. The product of each phase is a file the user keeps. Never skip Phase 0.

All outputs go to `deepfork-out/<target-name>/` in the current working directory.

## Phase 0 — License gate (non-negotiable)

Before cloning anything:

1. Check the target's license (`gh api repos/<owner>/<repo> --jq .license.spdx_id` or the LICENSE file).
2. **Proceed only for OSI-approved licenses** (MIT, Apache-2.0, BSD, GPL/AGPL/LGPL, MPL, etc.). No license = all-rights-reserved = **stop and tell the user** (you may still produce the *understanding* report from public code, but no rebuild).
3. State the rules of engagement in one line to the user:
   - The blueprint describes **behavior and architecture**, never copies code.
   - The rebuild is **clean-room**: written from the blueprint, not from the source.
   - Copyleft targets (GPL/AGPL): warn that a rebuild closely derived from the design may carry obligations; recommend the rebuild also be open source.
   - The final repo gets an `ATTRIBUTION.md` crediting the original.
4. Record license + rules in `deepfork-out/<target>/LICENSE-GATE.md`.

## Phase 1 — Acquire

```bash
git clone --depth 1 <target-url> deepfork-out/<target>/source
```

Note stars/age/activity in one line (context for how battle-tested the design is).

## Phase 2 — Comprehend (the graph pass)

**With graphify** (preferred — check `command -v graphify`; if missing, offer: `uv tool install graphifyy`, or proceed to the fallback):

```bash
cd deepfork-out/<target>/source
graphify .                          # code-only is free & local (tree-sitter AST)
# add --backend claude-cli if the repo's docs/images matter to understanding
graphify cluster-only .             # names communities, writes GRAPH_REPORT.md
graphify . --wiki                   # agent-crawlable wiki: index + one page per subsystem
```

**Fallback (no graphify):** build the map by hand — entry points (bin/main/exports), dependency manifest, directory tree with per-dir one-liners, and a call-graph sketch of the top 5 most-imported modules.

Then write **`UNDERSTANDING.md`** — the "most clean way to understand this repo":

1. **One-paragraph thesis** — what this repo actually is, mechanically.
2. **The load-bearing pieces** (graphify's god nodes): the 3-7 functions/classes everything routes through. For each: what it does, why everything depends on it.
3. **Subsystems** (the communities): one section each — purpose, key files, how it talks to the others.
4. **The data flow**: trace ONE representative request/call end to end, file by file.
5. **Surprising connections**: the non-obvious couplings (graphify's analysis pass surfaces these) — these are where rebuilds go wrong.
6. **Honesty labels**: mark each claim `[VERIFIED]` (you read the code) or `[INFERRED]` (from the graph/structure). Verify every load-bearing claim by reading the actual code before labeling it VERIFIED.

## Phase 3 — Interrogate

Answer the questions a rebuilder must know, using `graphify query / path / explain` (or direct reading):

- What is the core algorithm/trick, in plain words? (the thing you'd lose if you rebuilt naively)
- What do the public interfaces promise? (CLI surface, API contracts, file formats)
- Which dependencies are load-bearing vs incidental?
- What does it do at the edges — errors, concurrency, persistence?
- What would break first at 10× scale or hostile input?

Append answers to `UNDERSTANDING.md` § "Interrogation". Read the real code for anything still `[INFERRED]` that the rebuild depends on.

## Phase 4 — Blueprint

**Ask the user the customization question before writing** (this is the whole point — their version, not a clone):

> "What do you want YOUR version to do differently? (different language/stack, smaller scope, new capability, different storage, simpler/embedded, …)"

Then write **`BLUEPRINT.md`** — a spec someone could build from WITHOUT ever seeing the original source:

1. **Mission** — what the new tool does, for whom, in 2 sentences.
2. **Architecture** — components, responsibilities, boundaries (informed by the original's communities, *improved* where the surprising-connections pass showed accidental coupling).
3. **Core mechanisms** — each described **behaviorally** (inputs → transformation → outputs, invariants, edge rules). Pseudocode allowed; copied code forbidden.
4. **Contracts** — CLI/API surface, file formats, config schema.
5. **The user's customizations** — explicit deltas from the original's design, each with rationale.
6. **Build order** — milestones, each independently testable; test strategy per mechanism.
7. **Dependency decisions** — what to take, what to write, what to drop (with the original's choices as evidence).

## Phase 5 — Rebuild (clean-room)

1. Scaffold a fresh repo (new directory, user's name, their license choice).
2. **Close the original source.** Build from `BLUEPRINT.md` only. If you must check behavior, run the original as a black box (inputs/outputs) — do not read its code during implementation.
3. Build milestone by milestone, tests first for the core mechanisms.
4. Ship with `ATTRIBUTION.md`: "Design informed by reverse-engineering <original> (<license>). Implementation is original, built clean-room from a behavioral blueprint."
5. Final check: diff-of-spirit — does YOUR version do the user's deltas, not just clone the original?

## Guardrails

- Phase 0 always runs. Private/leaked/unlicensed code: understanding-report only, never a rebuild.
- Never paste original source into the rebuild. Behavioral description is the only thing that crosses the wall.
- If the target is enormous (>5k files), deepfork ONE subsystem the user picks from the community list.
- Keep `deepfork-out/` out of the rebuilt repo.
