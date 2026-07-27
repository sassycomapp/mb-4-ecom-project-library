---
document: README — project-library [mb4ecom]
doc-id: mb4ecom-0002
state: Live
date-created: 2026-07-27T115406+0200
---

# MB4 E-Commerce — Project Library

Planning, documentation, and specification repository for the MB4 E-Commerce project.

## Structure

| Folder | Purpose |
|---|---|
| `pre-project/` | Developer's first-thoughts, scoping ideas, ideation notes (input to pdlf) |
| `pdlf/` | pdlf orchestrator artifacts: devlog, ISA, BFT checkpoints |
| `pdlf/bft/` | BFT checkpoint output files per stage |
| `adr/` | Architecture Decision Records |
| `docs/` | Durable application documentation and specifications |
| `wireframes/` | HTML wireframes, mockups, visual layout references |
| `screens/` | Screen HTML mockups and compliance checklists |
| `wip/` | Active work-in-progress |

## pdlf

This project is managed using pdlf — a contiguous, stage-gated, memory-compounding
development strategy. Type `/pdlf` in OpenCode to init or resume.

**Code repo:** `../mb4ecom/` (Anvil-connected, GitHub VC)
**State file:** `~/.gstack/projects/mb4ecom/pdlf-state.md`
