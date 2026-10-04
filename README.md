# Agent Architecture Refactor

Architecture-as-Code refactor specification for migrating an existing LangGraph agent project to an explicit, traceable design-time architecture.

## Goal

Make these relationships machine-readable and visible:

```
Orchestrator → Route/Agent → Skill → Domain Knowledge → MCP → Tool
                                  ↓
                           Implementation → Eval
```

The architecture specification is the design-time source of truth. LangGraph remains the runtime implementation. React Flow is the visualization layer, not the source of truth.

## Reference projects

- Datalayer AgentSpecs: https://github.com/datalayer/agentspecs
- LangGraph: https://github.com/langchain-ai/langgraph
- React Flow / xyflow: https://github.com/xyflow/xyflow

## Start here

1. Read `AGENTS.md`.
2. Read `docs/refactor-plan.md`.
3. Inventory the actual source LangGraph repository/branch before modifying runtime code.
4. Complete Phase 0 before implementation.
5. Use `architecture/README.md` as the contract for the canonical design graph.

The primary acceptance test is document-driven refactoring: change one domain rule, deterministically identify affected routes/skills/implementation/evals, refactor the runtime, run evals, and verify the design-time graph updates.
