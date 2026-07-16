# Opsight Intelligence Platform — Architecture Overview

> A public, high-level overview of the Opsight Intelligence Platform.
> Implementation repositories are private; this document describes the
> architecture and design principles, not internal details.

Opsight is a **federated multi-domain intelligence platform**. Independent
domain engines — spanning areas such as fraud, maritime, manufacturing, and
AI-governance — each run their own **collect → assess** pipeline and publish a
single, standardized **intelligence contract** to a shared bus. A federation
gateway then exposes these assessments to executive-facing AI personas.

The guiding idea is **contract-first architecture**: one shared schema, strict
separation between pure business logic and I/O, and engines that stay fully
independent while speaking the same language.

---

## Design principles

- **Contract-first** — every engine publishes to one shared, versioned schema,
  so domains evolve independently without breaking consumers.
- **Pure core, thin I/O** — business logic is isolated from adapters
  (databases, queues, external APIs), keeping it testable and portable.
- **Dependency inversion** — the core depends on abstractions; adapters
  implement them, never the other way around.
- **Idempotent, content-addressed artifacts** — reprocessing the same input
  yields the same result, safely.
- **Single source of truth** — shared registries instead of duplicated config.

---

## System shape (conceptual)

```
                    ┌─────────────────────────────────────┐
                    │        Executive AI Personas        │
                    │  (consume structured assessments)   │
                    └──────────────────┬──────────────────┘
                                       │
                    ┌──────────────────┴──────────────────┐
                    │        Federation Gateway (MCP)      │
                    └──────────────────┬──────────────────┘
                                       │
                    ┌──────────────────┴──────────────────┐
                    │           Intelligence Bus           │
                    │   (standardized intelligence contract)│
                    └───┬───────────┬───────────┬──────────┘
                        │           │           │
                 ┌──────┴───┐ ┌─────┴────┐ ┌────┴─────┐
                 │  Domain  │ │  Domain  │ │  Domain  │   ... independent
                 │  Engine  │ │  Engine  │ │  Engine  │       engines
                 └──────────┘ └──────────┘ └──────────┘
             each: collect → normalize → enrich → detect → score → assess → publish
```

---

## Engine pipeline

Every domain engine follows the same stages, so the platform stays uniform
even as domains differ:

`collect → normalize → enrich → detect → score → assess → publish`

Each stage is small, typed, and independently testable. Engines never talk to
each other directly — they only publish to the shared bus.

---

## Tech

Python · FastAPI · Pydantic · PostgreSQL · Snowflake · Docker · MCP ·
LLM agents · GitHub Actions · pytest · Ruff

---

## Engineering discipline

- Small modules with full type hints
- Real CI gates: lint + types + tests
- Reproducible releases
- Clear boundaries between pure logic and I/O

---

*This overview intentionally omits implementation details. For questions about
the architecture, feel free to reach out.*
