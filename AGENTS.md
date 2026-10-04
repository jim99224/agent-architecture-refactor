# AGENTS.md

## Purpose

This repository defines the source-of-truth architecture and migration rules for an existing LangGraph project.

## Source-of-truth hierarchy

Treat these as design-time sources of truth:

1. `architecture/` — machine-readable architecture specifications and schemas.
2. `domains/` — human-maintained domain rules with stable IDs.
3. `skills/*/SKILL.md` — reusable capability contracts.
4. Runtime code implements these specifications; it must not redefine them independently.

## Required workflow

When architecture/domain/skill specifications change:

1. validate architecture;
2. calculate deterministic impact;
3. inspect affected implementation;
4. modify runtime implementation when required;
5. update affected tests and evals;
6. compile the canonical graph;
7. verify the frontend representation;
8. run regression tests.

Never modify architecture documents merely to make incorrect existing implementation pass.

## Refactor rules

- Inventory the existing LangGraph system before changing behavior.
- Preserve current supported runtime behavior unless a specification explicitly changes it.
- Do not equate every LangGraph node or Python function with a Skill.
- Do not use an LLM as the primary dependency resolver.
- Do not make React Flow JSON the canonical architecture model.
- Do not duplicate architecture manually in frontend code.
- Prefer stable IDs and explicit references over filename inference.
- Keep migration incremental; every phase should leave the project runnable.
- Run relevant tests before claiming a phase is complete.

## External references

Study, but do not blindly copy:
- https://github.com/datalayer/agentspecs
- https://github.com/langchain-ai/langgraph
- https://github.com/xyflow/xyflow

Record material design deviations in ADRs.
