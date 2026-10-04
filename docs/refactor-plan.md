# LangGraph → Architecture-as-Code Refactor Plan

## Mission

Refactor the existing LangGraph project so architecture is explicit, machine-readable, visualizable, testable, and traceable. LangGraph remains the runtime unless a specific replacement is justified. The design-time specification becomes the source of truth.

The system must expose Orchestrator, Agent/Subagent, Route, Skill, Domain Knowledge, MCP Server, Tool, Prompt/Policy, Implementation and Eval/Test relationships.

## 1. Reference implementations

Before coding, inspect current versions of:

- `datalayer/agentspecs`: study agents, teams, skills, MCP servers, tools, evals, versioned YAML references and generated Python/TypeScript catalogs. Adopt useful concepts but extend them where domain-rule and implementation traceability are missing.
- `langchain-ai/langgraph`: preserve it as the runtime orchestration layer. Do not rewrite working behavior merely to match the design representation.
- `xyflow/xyflow`: use React + TypeScript + `@xyflow/react` for the design-time explorer unless a blocking limitation is documented.

Write an ADR covering what was adopted/adapted and why.

## 2. Phase 0 — discovery first

Do not refactor immediately. Inspect the actual source repository and the branch containing the current LangGraph implementation.

Inventory:
- graphs, subgraphs, nodes and conditional edges;
- routers and orchestration decisions;
- agents/subagents;
- prompts/policies;
- tools and MCP integrations;
- retrieval/repository-search components;
- state/checkpoint/persistence;
- domain-specific rules embedded in Python/prompts;
- API entrypoints;
- tests and evals.

Produce `docs/current-architecture.md` and `architecture/discovery/current-architecture.json`.

Classify behavior as runtime mechanism, orchestration decision, domain knowledge, skill, tool, infrastructure, prompt/policy, or evaluation. No behavior changes in Phase 0.

## 3. Target structure

Prefer, but do not force:

```
AGENTS.md
architecture/
  agents/
  orchestrators/
  skills/
  mcp-servers/
  tools/
  evals/
  domains/
  schemas/
  generated/
domains/
skills/<skill>/SKILL.md
src/
  runtime/langgraph/
  architecture/{parser,resolver,validator,compiler,impact}/
frontend/
evals/
tests/
docs/
```

Document justified deviations.

## 4. Architecture specification

Use declarative YAML inspired by AgentSpecs. Every independently addressable entity/rule must have a stable ID. Prefer versioned references where practical. Cross-references must be schema validated.

Example:

```yaml
id: source-code-investigation
version: 0.1.0
type: route
domain_rules: [CODE-001, CODE-002]
skills:
  - knowledge-search:0.1.0
  - repo-search:0.1.0
implementation:
  - src/runtime/langgraph/router.py
evals:
  - code-search-routing:0.1.0
```

## 5. Domain knowledge

Do not leave changeable domain decisions only as Python branching logic or opaque prompts. Put human-maintained rules under `domains/` and give each rule a stable ID.

Example: `CODE-001` states that implementation-specific questions require current source evidence, not knowledge retrieval alone.

Architecture, implementation and evals must be traceable to that ID.

## 6. Skills

A Skill is a reusable capability, not every LangGraph node. A `SKILL.md` should expose ID, purpose, inputs/outputs, domain-rule dependencies, MCP/tool dependencies, implementation binding and eval dependencies.

## 7. Architecture compiler

Implement a compiler pipeline:

```
Specs/docs → Parse → Validate → Resolve → Canonical IR → Graph/Impact outputs
```

It must:
- validate schemas;
- detect duplicate IDs and dangling references;
- resolve versions and Skill→MCP→Tool relationships;
- resolve Domain Rule→Skill/Route relationships;
- resolve architecture→implementation/eval mappings;
- emit deterministic canonical IR;
- emit frontend graph JSON through an adapter;
- support deterministic impact analysis.

Do not couple Canonical IR directly to React Flow.

Provide CLI behavior equivalent to:

```
architecture validate
architecture compile
architecture graph
architecture impact --base main --head HEAD
```

Validation must fail CI for invalid schema, duplicate IDs, dangling refs, invalid versions, required missing implementation/eval bindings, and unresolved Skill/MCP/Tool refs.

## 8. Runtime mapping

Create deterministic traceability between design entities and LangGraph implementation. An annotation/registry such as `@implements("route:source-code-investigation")` is acceptable, but filename convention alone is not.

The design spec describes intent; LangGraph implements it.

## 9. Frontend

Build a React + TypeScript + React Flow Design-time Architecture Explorer. The frontend reads generated graph data and MUST NOT parse YAML or hard-code a second architecture copy.

Default hierarchy:

```
Orchestrator → Route/Agent → Skill → Domain Knowledge → MCP → Tool
```

Implementation and Eval nodes may be collapsed initially.

Clicking a node opens a detail panel. Domain Rule details include ID, description, source document, routes/skills using it, implementation, evals, validation status and Git location. Skill details include domain knowledge, MCP/tools, implementation and evals. Implementation details include file/symbol and what it implements.

Support node-type filtering, graph search, expand/collapse and a critical **Domain Logic View**:

```
Domain Rule → Skill → Route → Implementation → Eval
```

Searching `CODE-001` or `repo-search` must locate the node and navigate its relationships.

## 10. Impact analysis

Given a changed stable ID, traverse the canonical graph and return affected routes, skills, implementations and evals. An LLM may explain results but cannot invent dependencies.

For Git diff impact:

```
Git diff
→ changed architecture/domain files
→ stable IDs
→ graph traversal
→ affected components
```

Return human-readable and JSON output.

## 11. Evals and regression

Important domain rules should have evals where practical.

Example:

```yaml
id: CODE-001-E01
validates: [CODE-001]
input:
  question: "Where is JWT validation implemented?"
expect:
  required_skills: [knowledge-search, repo-search]
  source_evidence_required: true
```

Before runtime refactoring, establish regression tests for existing supported behavior. Where current behavior is ambiguous, document rather than guess.

CI order:

```
architecture validation
→ unit/regression tests
→ architecture consistency tests
→ frontend tests
→ eval smoke tests
```

Compilation must be reproducible and must not require an LLM.

## 12. Incremental migration

A. Inventory current LangGraph system; no behavior change.
B. Introduce schemas and Canonical IR.
C. Describe CURRENT runtime declaratively before redesigning it.
D. Introduce domain-rule IDs and traceability.
E. Add compiler/validation.
F. Add implementation/eval mapping.
G. Add React Flow frontend.
H. Add impact analysis.
I. Enable document-driven refactoring.

Every phase must leave the project runnable.

## 13. Acceptance criteria

**AC-01 Deterministic compile:** valid specs compile to semantically identical canonical graph output across repeated runs.

**AC-02 Broken reference:** `repo-serach` fails validation with an actionable unknown-skill error and source reference.

**AC-03 Domain traceability:** from `CODE-001`, deterministically find domain document → route/skill → implementation → eval without LLM inference.

**AC-04 MCP traceability:** from `repo-search`, show MCP and tools.

**AC-05 Frontend source:** frontend displays compiler-generated architecture; no hard-coded duplicate graph.

**AC-06 Node details:** clicking `CODE-001` shows source, consumers, implementation and evals.

**AC-07 Domain Logic View:** clearly shows Domain Rule → Skill → Route → Implementation → Eval.

**AC-08 Impact:** changing `CODE-001` identifies affected implementation/evals.

**AC-09 Runtime compatibility:** existing supported LangGraph workflows continue to run.

**AC-10 Consistency:** automated test verifies required design routes/skills have runtime implementations.

**AC-11 CI:** dangling architecture references fail CI.

**AC-12 End-to-end proof:** perform and document one real exercise: modify a domain rule → run impact → identify affected runtime → refactor → update/run eval → compile → verify frontend graph reflects the change.

AC-12 is the primary proof of success.

## 14. Non-goals

Do not replace LangGraph without cause; confuse runtime traces with design architecture; make every Python function a node; use LLM inference for deterministic dependencies; make React Flow JSON source-of-truth; duplicate architecture in frontend; put all prose domain knowledge into YAML; or perform a big-bang rewrite.

## 15. Required deliverables

At completion provide:

```
docs/current-architecture.md
docs/target-architecture.md
docs/migration-report.md
docs/acceptance-report.md
docs/adr/*
architecture/schemas/*
architecture/generated/graph.json
domains/*
skills/*/SKILL.md
src/architecture/{parser,validator,resolver,compiler,impact}/*
frontend/*
evals/*
AGENTS.md
```

## Definition of Done

The following must work end-to-end:

```
Developer changes CODE-001
→ architecture detects the changed stable ID
→ impact engine finds affected Skill/Route
→ finds implementation and tests/evals
→ coding agent receives deterministic scope
→ runtime is refactored
→ tests/evals pass
→ compiler regenerates graph
→ design-time UI reflects the new architecture
```

If this cannot be demonstrated, the refactor is not complete.

## Coding-agent execution protocol

Before writing code: inspect the source repo, inventory current architecture, compare it with this target, propose phased changes, identify migration risks and behavior that must remain compatible.

For every implementation phase report:
- changed architecture;
- changed implementation;
- changed tests;
- changed evals;
- migration risks;
- commands executed;
- actual results.

Never claim success without executing relevant validation/tests.
