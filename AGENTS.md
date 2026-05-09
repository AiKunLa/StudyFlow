# StudyFlow - Root AGENTS.md

This file provides project-level guidance for contributors and coding agents working on StudyFlow.

## Project Overview

**StudyFlow** is a material-driven AI learning loop product. The MVP focuses on turning a user's own study material into a short, verifiable learning loop:

1. Import study material
2. Segment it into learning units
3. Explain and answer questions based on the source text
4. Run a lightweight quiz
5. Generate a follow-up review task

**Primary users**:

- Self-directed knowledge workers
- Certificate and exam learners

**Key product values**:

- Source-grounded AI assistance
- Low-friction learning flow
- Verifiable short-loop outcomes
- Progressive expansion instead of all-at-once intelligence claims

## Current Repository Stage

This repository is still in the initialization phase. It currently contains requirement, risk, and planning documentation, but no application code yet.

That means most work at this stage falls into three buckets:

- Tightening product scope
- Establishing implementation boundaries
- Converting requirements into an executable build plan

## Planned Architecture

The implementation is not scaffolded yet, but the intended system shape is:

```text
StudyFlow
├── docs/                    Documentation entry tree
│   ├── index.md            Index of docs subdirectories
│   ├── product/            Product definitions and scope
│   │   └── index.md
│   └── engineering/        Architecture and delivery planning
│       └── index.md
├── apps/
│   ├── web/                User-facing application
│   └── api/                API and orchestration layer
└── packages/
    ├── domain/             Core entities and state transitions
    ├── ingestion/          Material import and segmentation
    ├── ai/                 Explanation, Q&A, quiz generation
    └── scheduler/          Review task scheduling
```

Until the codebase exists, keep architecture decisions aligned with these ownership boundaries rather than mixing everything into one application layer.

## Useful Sources

- [README.md](D:/Code/VSCode_Project/StudyFlow/README.md)
- [docs/index.md](D:/Code/VSCode_Project/StudyFlow/docs/index.md)
- [AI学习闭环系统-MVP需求文档.md](D:/Code/VSCode_Project/StudyFlow/AI学习闭环系统-MVP需求文档.md)
- [AI学习闭环系统-问题与风险清单.md](D:/Code/VSCode_Project/StudyFlow/AI学习闭环系统-问题与风险清单.md)

## Product Constraints

The MVP must preserve these constraints:

- AI explanations and Q&A should stay grounded in the current learning unit or current project material.
- The MVP proves a short learning loop, not a complete long-term learning operating system.
- Mastery is represented as discrete states, not pseudo-precise scores.
- Learning unit segmentation only needs to be usable and readable, not a perfect knowledge graph.
- Quiz design emphasizes understanding and recall, not full exam simulation.
- The main user path should remain clear: import -> study -> explain or ask -> quiz -> review.

## Important Non-Goals

Do not quietly reintroduce these as default MVP scope:

- Web-scale material search
- High-confidence knowledge dependency graphs
- Complex long-term planning systems
- Precise mastery scoring
- Heavy collaboration or social features

## Project Constraints

The repository must preserve these structural constraints:

- Documentation uses progressive disclosure by directory, not a flat file list.
- `docs/index.md` only indexes directories under `docs/`.
- Every subdirectory under `docs/` must contain its own `index.md`.
- A parent index should point to child directory indexes before pointing to leaf documents.
- When adding a new docs directory, its `index.md` must be added in the same change.

## Documentation Structure

This repository uses progressive disclosure for docs:

- `docs/index.md` indexes directories under `docs/`
- each subdirectory under `docs/` must contain its own `index.md`
- directory indexes should point to local files in that directory
- root-level navigation should point to directory indexes before pointing to leaf documents

When adding a new docs directory, add its `index.md` in the same change.

## Component References

Current documentation entry points:

- [docs/index.md](D:/Code/VSCode_Project/StudyFlow/docs/index.md): index of documentation areas
- [docs/product/index.md](D:/Code/VSCode_Project/StudyFlow/docs/product/index.md): product scope and MVP definition
- [docs/engineering/index.md](D:/Code/VSCode_Project/StudyFlow/docs/engineering/index.md): architecture and implementation planning

Future code-local guides should be added here once those directories exist:

- `apps/web/AGENTS.md`
- `apps/api/AGENTS.md`
- `packages/domain/AGENTS.md`
- `packages/ingestion/AGENTS.md`
- `packages/ai/AGENTS.md`
- `packages/scheduler/AGENTS.md`

## Documentation Map

- [README.md](D:/Code/VSCode_Project/StudyFlow/README.md): human-facing project overview
- [docs/index.md](D:/Code/VSCode_Project/StudyFlow/docs/index.md): docs root index
- [docs/product/index.md](D:/Code/VSCode_Project/StudyFlow/docs/product/index.md): product document index
- [docs/engineering/index.md](D:/Code/VSCode_Project/StudyFlow/docs/engineering/index.md): engineering document index

## Testing Strategy

There is no runnable code or test suite yet.

For documentation work, verify:

- links resolve correctly
- root docs point to directory indexes
- each docs directory contains an `index.md`
- parent docs do not duplicate child-level details unnecessarily

## Common Tasks

### Refine Product Scope

1. Update [docs/product/mvp-definition.md](D:/Code/VSCode_Project/StudyFlow/docs/product/mvp-definition.md)
2. If scope changes affect system shape, update [docs/engineering/architecture.md](D:/Code/VSCode_Project/StudyFlow/docs/engineering/architecture.md)
3. If delivery order changes, update [docs/engineering/implementation-plan.md](D:/Code/VSCode_Project/StudyFlow/docs/engineering/implementation-plan.md)

### Add A New Docs Area

1. Create a new subdirectory under `docs/`
2. Add that directory's `index.md`
3. Add the new directory index to [docs/index.md](D:/Code/VSCode_Project/StudyFlow/docs/index.md)
4. Only then add leaf documents inside that directory

### Scaffold The First Codebase

1. Create directories by bounded responsibility, not by scattered pages or endpoints
2. Add a local `AGENTS.md` when a subsystem directory becomes stable
3. Keep local `AGENTS.md` files focused on local rules, not root-level repetition

## Important Quirks & Gotchas

- Do not write architecture as if implementation already exists when the repo is still pre-code.
- Do not let leaf docs become the only source of truth; directory indexes must remain navigable.
- Do not bypass directory indexes by growing `docs/index.md` into a flat file catalog.
- If a technical decision becomes stable, record it in `docs/engineering/` instead of scattering it across root docs.
