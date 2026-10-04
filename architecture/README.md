# Architecture Contract

## Canonical pipeline

```
YAML / Markdown / SKILL.md
          ↓
        Parser
          ↓
       Validator
          ↓
       Resolver
          ↓
    Canonical IR
          ↓
 Architecture Graph
     ↓          ↓
 graph.json   impact data
     ↓
 React Flow adapter
     ↓
 Design-time UI
```

The compiler is more than YAML-to-JSON. It validates schemas, resolves references, detects duplicate/dangling IDs, builds semantic relationships, maps architecture to implementation/evals, and supports impact analysis.

## Minimum node types

- orchestrator
- agent
- route
- skill
- domain_rule
- domain_document
- mcp_server
- tool
- prompt
- implementation
- eval

## Minimum edge types

- ROUTES_TO
- USES_SKILL
- USES_KNOWLEDGE
- USES_MCP
- USES_TOOL
- IMPLEMENTED_BY
- VALIDATED_BY
- DEPENDS_ON

## Important boundary

The Canonical IR is framework-neutral. React Flow receives an adapted `nodes[]/edges[]` representation. LangGraph runtime edges and runtime traces must not be confused with the design-time semantic graph.

## Example

```yaml
id: source-code-investigation
version: 0.1.0
type: route
domain_rules:
  - CODE-001
skills:
  - knowledge-search:0.1.0
  - repo-search:0.1.0
implementation:
  - src/runtime/langgraph/router.py
evals:
  - code-search-routing:0.1.0
```

A domain document may define:

```markdown
## CODE-001 — Implementation questions require source evidence

Implementation-specific questions must obtain evidence from the current source repository; knowledge retrieval alone is insufficient.
```

The graph must be able to traverse CODE-001 → route/skill → implementation → eval without LLM inference.
