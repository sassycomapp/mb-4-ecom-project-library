# AGENTS.md — project-library [mb4ecom]

## Project purpose
This is the planning, documentation, and specification repository for the MB4 E-Commerce
application (mb4ecom). It is not the live code repository.

Its purpose is to:
- produce validated documentation, specifications, and ADRs,
- house wireframes and design artefacts that guide Anvil UI development,
- store bdlf orchestrator artifacts (devlog, ISA, BFT checkpoints),
- drive the bdlf development workflow for mb4ecom.

The live application code lives in `../mb4ecom/`. Read from it for context when needed.
Do not treat this repo as the application implementation workspace.

## Hard rules (always apply)

### 1. anvil.yaml protection

Hard rule: never ever write to anvil.yaml.

- Treat anvil.yaml as Anvil-managed metadata/configuration.
- Do not create, edit, reformat, regenerate, normalize, merge-edit, or patch it.
- Do not include it in bulk refactors or automated file updates.
- If a task appears to require changing anvil.yaml, stop and ask the user for explicit approval.
- Assume Anvil is the authoritative writer for anvil.yaml.

### 2. Two-repo separation

- This repo (`project-library/`) = planning, docs, ADRs, wireframes, bdlf artifacts. Does NOT go to Anvil.
- Code repo (`../mb4ecom/`) = live Anvil code + anvil.yaml. Goes to Anvil via GitHub VC.
- The agent writes to project-library freely. The agent writes to the code repo only during
  bdlf stages 14–16 (frontend assembly, feature development, integration) and only for code
  files — never anvil.yaml.

### 3. GBrain cross-contamination discipline

GBrain stores pages from multiple projects. To prevent cross-project contamination:

- **When querying GBrain**, always tag every result with its source project.
- State explicitly: "this result is from project X — advisory, not canonical — confirm before applying."
- Cross-project **learnings** are welcome (a mistake in one project can inform another).
- Cross-project **decisions** (ADRs, ISAs, architecture) are NOT transferable unless the human
  explicitly confirms.
- When possible, scope queries using `--source mb4ecom` to restrict results to this project.
- At Junction 0 (LEARN), query both this project's source AND the default source for prior
  learnings, but clearly separate "this project's prior art" from "other projects' learnings."

### 4. bdlf artifact routing

| Artifact | Destination |
|---|---|
| Devlog, ISA, BFT checkpoints | `bdlf/` |
| ADRs | `adr/` |
| Wireframes, mockups | `wireframes/` |
| Screen HTML mockups | `screens/` |
| Durable docs | `docs/` |
| WIP files | `wip/` |
| Pre-project first-thoughts | `pre-project/` |
| State file | `~/.gstack/projects/mb4ecom/bdlf-state.md` |

### 5. Fact rule and certainty levels

- Do not use general training data or prior memory to assert project-specific facts.
- If a fact is not supported by files in this repo or the code repo, treat it as inferred or missing.
- Every statement must fall into one of: Clearly defined, Partially defined, Inferred only, Missing.
- Do not upgrade inferred material to clearly defined.

## Workspace structure

| Folder | Purpose |
|---|---|
| `pre-project/` | Developer's first-thoughts, scoping ideas, ideation notes (input to bdlf) |
| `bdlf/` | bdlf orchestrator artifacts: devlog, ISA, BFT checkpoints |
| `bdlf/bft/` | BFT checkpoint output files per stage |
| `adr/` | Architecture Decision Records — one decision per file |
| `docs/` | Durable application documentation, specs, UI standards, DESIGN.md |
| `wireframes/` | HTML wireframes, mockups, and visual layout references |
| `screens/` | Screen HTML mockups and their compliance checklists |
| `wip/` | Active work-in-progress: current plans, prompts, and files being prepared |

## bdlf workflow

This project is managed using bdlf — a contiguous, stage-gated, memory-compounding
development strategy. Type `/bdlf` in OpenCode to init or resume.

- **bdlf home:** `C:\bdlf\` (skill, templates, registry — shared across all projects)
- **State file:** `~/.gstack/projects/mb4ecom/bdlf-state.md` (machine resume token)
- **Devlog:** `bdlf/devlog.md` (human append-only audit trail)
- **ISA:** `bdlf/isa.md` (living done-definition)
- **GBrain source:** `mb4ecom` (isolated — scoped queries only)

## Safety rules

- Do not create, edit, move, or delete files unless the user clearly asks for that operation.
- Never run destructive operations without a clear warning and explicit confirmation.
- Prefer proposing diffs or patches in chat over auto-applying changes.
- Git operations are permission-only.
- Do not push, publish, sync, or export anything unless the user explicitly asks.

## Communication style

- Keep answers concise, structured, and practical.
- Distinguish clearly between facts from the repo, inferences, and proposed revisions.
- Flag uncertainty, contradictions, or missing provenance explicitly.
