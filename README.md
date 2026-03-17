# Agentic Control Kernel

A unifying control-systems metalayer for LLM-as-controller agent development.

> **Core law**: Do not grant an agent more mutation freedom than your evaluator can reliably judge.

## What Is This?

A purely knowledge-based **skill** that bootstraps any repository with the full control-systems worldview for autonomous agent development. When installed, it provides:

- **Typed JSON schemas** for plant state, control directives, trace ledger, and evaluator outputs
- **Safety shield conventions** (CBF-QP, policy gates, containment invariants)
- **Multi-rate loop hierarchy** defining where LLMs should and shouldn't operate
- **Plant interface API specs** for physical, cyber-physical, and cyber systems
- **EGRI problem-spec templates** for evaluator-governed controller improvement
- **Consciousness stack integration** for persistent cross-session agent memory

## Architecture

The LLM emits typed **control directives** (θ_t) — not raw actuations (u_t). Deterministic controller modules execute, safety shields filter, and the runtime logs traces to an append-only ledger.

```
Plant → observe() → Runtime → estimator → belief state
  → LLM: decision → θ_t (typed directive)
  → Controller: propose → proposed u_t
  → Safety Shield: filter → safe u_t + certificate
  → Plant: apply → result
  → Ledger: trace + score
```

## Synthesized Subsystems

| Layer | Source | Role |
|-------|--------|------|
| Governance | [control-metalayer-loop](https://github.com/broomva/control-metalayer) | Setpoints, sensors, gates, policy, profiles |
| Improvement | [autoany](https://github.com/broomva/autoany) (EGRI) | Evaluator-governed recursive improvement |
| Orchestration | [symphony](https://github.com/broomva/symphony) | Poll/dispatch/worker/reconcile daemon (Rust) |
| Episodic Memory | knowledge-graph-memory | Conversation logs → Obsidian bridge |
| Consciousness | agent-consciousness | Three-substrate persistent context |
| QA/Actuation | gstack | Headless browser, workflow skills |
| **Control Kernel** | **this repo** | Plant interface, safety shields, typed schemas |

## Quick Start

### Bootstrap a project

```bash
python3 scripts/control_kernel_init.py <repo-path> [--profile governed]
```

Installs into target repo:
- `.control/policy.yaml` — setpoints, gates, profiles
- `.control/plant.yaml` — typed state/action definitions
- `schemas/` — JSON schemas for state, action, trace, evaluator
- `METALAYER.md` — control loop definition
- `problem-spec.control.yaml` — EGRI template for controller optimization

### Install as a skill

```bash
# The packaged .skill file can be installed into any Claude Code environment
# See SKILL.md for full trigger conditions and usage
```

## Documentation

| Document | Description |
|----------|-------------|
| [SKILL.md](SKILL.md) | Skill entry point — triggers, workflow, quick start |
| [architecture.md](references/architecture.md) | 5-layer stack, formal control law, method→component mapping |
| [plant-interface.md](references/plant-interface.md) | Plant/Estimator/Controller/Shield/Evaluator API specs |
| [safety-shields.md](references/safety-shields.md) | CBF-QP, policy gates, containment, failure modes |
| [multi-rate-hierarchy.md](references/multi-rate-hierarchy.md) | Loop rates, LLM placement heuristics |
| [world-models.md](references/world-models.md) | Koopman, DeePC, digital twins, learned dynamics |
| [egri-for-controllers.md](references/egri-for-controllers.md) | Autoany applied to controller optimization |
| [orchestration-patterns.md](references/orchestration-patterns.md) | Symphony daemon patterns |
| [consciousness-stack.md](references/consciousness-stack.md) | Memory/knowledge/episodic integration |
| [failure-modes.md](references/failure-modes.md) | Mitigations catalog |
| [deep-research-report.md](references/deep-research-report.md) | Original research report and project plan |

## Schemas

| Schema | Purpose |
|--------|---------|
| [state.schema.json](schemas/state.schema.json) | Plant/belief state (measured, estimated, context) |
| [action.schema.json](schemas/action.schema.json) | Control directives θ_t (setpoints, mode switches, params) |
| [trace.schema.json](schemas/trace.schema.json) | Ledger entries (autoany-compatible, with shield certs) |
| [evaluator.schema.json](schemas/evaluator.schema.json) | Score vectors, promotion decisions, scenario coverage |

## Design Decisions

- **Skill = knowledge** (stateless templates that shape reasoning), **Runtime = execution** (symphony daemon, gstack browser, MPC solvers)
- LLMs output **controller parameters and plans**, not raw actuations — except in slow cyber plants
- Safety is two-layered: **policy gates** (semantic) + **CBF-QP shields** (dynamic) — neither alone suffices
- All improvement is **evaluator-governed** (EGRI): freeze harness, mutate surface, evaluate, promote/rollback

## License

Apache 2.0
