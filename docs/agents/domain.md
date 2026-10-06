---
title: "domain"
document: Domain Docs
doc-id: "mb-7efb8e65"
updated: "2026-10-06"
state: Live
date-created: 2026-07-25T214644+0200
---
# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Two-repo architecture

This application spans two repos:

- **`mb4ecom-project-library`** (this repo) — planning, documentation, ADRs, wireframes, design artefacts. PDLF-managed.
- **`mb4ecom`** — live application code (Python/Anvil). Built on Anvil.works.

Both repos apply to the same application. When a skill needs implementation context, read from `mb4ecom`. When it needs architectural decisions or design context, read from this repo.

## Before exploring, read these

- **`CONTEXT.md`** at the repo root — domain glossary and key concepts (create lazily via `/domain-modeling` when needed).
- **`adr/`** at the repo root — Architecture Decision Records.

If any of these files don't exist, **proceed silently**. The `/domain-modeling` skill (reached via `/grill-with-docs` and `/improve-codebase-architecture`) creates them lazily when terms or decisions actually get resolved.

## File structure

Single-context repo:

```
/
├── CONTEXT.md                ← domain glossary (created lazily)
├── adr/                      ← Architecture Decision Records
├── docs/
│   └── agents/               ← Agent configuration
├── pre-project/              ← Developer's first-thoughts (PDLF input)
├── pdlf/                     ← PDLF orchestrator artifacts
├── wireframes/               ← HTML wireframes
├── screens/                  ← Screen HTML mockups
└── wip/                      ← Work in progress
```

## Use the glossary's vocabulary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in `CONTEXT.md`. Don't drift to synonyms the glossary explicitly avoids.

## Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding.
