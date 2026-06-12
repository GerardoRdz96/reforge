# reforge

**Understand any repo. Rebuild it yours.**

reforge is an agent skill that reverse-engineers any open-source repository into (1) the cleanest possible explanation of how it actually works, and (2) a behavioral blueprint you can rebuild from — with your own changes, in your own stack, clean-room.

Stop reading 60k lines to understand a tool. Stop forking when what you wanted was *your own version*.

```
you:    /reforge https://github.com/karpathy/micrograd — but I want it in TypeScript with a graph visualizer
agent:  → license gate ✓ MIT
        → knowledge graph: 47 nodes, 4 subsystems, 2 god nodes (Value, MLP)
        → UNDERSTANDING.md   the repo, explained clean — load-bearing pieces, data flow, the core trick
        → BLUEPRINT.md       a spec you could build from without ever seeing the source
        → rebuild/           your TypeScript version, built clean-room from the blueprint, tests first
```

## Install

Works with Claude Code (and any agent that reads [skills](https://github.com/anthropics/skills)):

```bash
npx skills add GerardoRdz96/reforge
```

Optional but recommended — the graph engine that makes the understanding pass exceptional ([graphify](https://github.com/safishamsi/graphify), 65k★):

```bash
uv tool install graphifyy   # double-y; code analysis is local & free (tree-sitter)
```

Without graphify, reforge falls back to manual repo mapping. With it, you get god-node detection, auto-named subsystems, surprising-connection analysis, and token-cheap graph queries.

## What you get

| Artifact | What it is |
|---|---|
| `UNDERSTANDING.md` | The repo explained the way you wish its docs did: the 5 load-bearing pieces, each subsystem, one request traced end-to-end, the non-obvious couplings. Every claim labeled `[VERIFIED]` or `[INFERRED]`. |
| `BLUEPRINT.md` | A behavioral spec — mechanisms, contracts, build order, test strategy — plus **your customization deltas**. Someone who never saw the original could build from it. That someone is your agent. |
| `rebuild/` | Your version. Clean-room: built from the blueprint with the original source closed. Ships with `ATTRIBUTION.md`. |

## Worked example

[`examples/micrograd/`](examples/micrograd/) — karpathy's micrograd (12k★) reforged: the full understanding report, the blueprint, and what a customized rebuild looks like. Generated end-to-end by the skill, lightly trimmed.

## Why clean-room?

reforge is built to keep you on the right side of open source:

- **Phase 0 license gate** — it checks the target's license before anything else and refuses rebuilds of unlicensed code.
- **The blueprint wall** — only *behavioral descriptions* cross from the original to your rebuild. Never code. Your implementation is original work.
- **Copyleft awareness** — GPL/AGPL targets come with a warning and a recommendation that your rebuild stay open.
- **Attribution by default** — every rebuild credits the original design.

This is how engineers have legally reimplemented systems for decades (Compaq vs IBM BIOS, 1982). reforge just makes the discipline automatic.

## How it works

1. **License gate** — SPDX check, rules of engagement stated up front.
2. **Comprehend** — graphify builds a knowledge graph (locally, free for code); the skill turns god nodes + communities + surprising connections into `UNDERSTANDING.md`.
3. **Interrogate** — the agent answers the questions a rebuilder must know (the core trick, the contracts, what breaks at 10×), verifying inferred claims against real code.
4. **Blueprint** — asks what YOU want different, then writes the spec with your deltas designed in.
5. **Rebuild** — fresh repo, original closed, blueprint only, tests first.

## FAQ

**Is this just "fork it"?** No. A fork keeps their code, their architecture, their language, their debt. reforge gives you their *lessons* in a spec, and a version that's actually yours.

**Is this legal?** Understanding public code is legal everywhere. Clean-room reimplementation from a behavioral spec is the industry-standard legal path. The license gate + blueprint wall keep the discipline honest. (Not legal advice; if you're rebuilding something commercial-sensitive, ask a lawyer.)

**Does it work on huge repos?** Yes — pick one subsystem from the community list and reforge that. The graph makes subsystem boundaries visible.

**Which agents?** Claude Code first-class. The skill is plain markdown — Codex, Cursor, Gemini CLI and friends can run it too.

---

MIT · Built by [The Penguin Alley](https://penguinalley.com) · Powered by [graphify](https://github.com/safishamsi/graphify)
