# ADR-003: Agent Skills Are Out of Scope

## Status

Accepted

## Context

During specification development, we considered adding Skill and Skillset as new resource kinds to the AI Resource Specification. The Agent Skills format (https://agentskills.io) defines a standard for packaging AI agent capabilities as directories containing instructions, scripts, and resources.

### The Question

Should the AI Resource Specification include Skill and Skillset resource kinds that either:
1. Inline the Agent Skills format into YAML
2. Reference Agent Skills directories
3. Provide an abstraction layer over Agent Skills

### Analysis

**Agent Skills landscape:**
- Established open standard with clear specification
- Wide adoption: Claude, Cursor, VS Code, GitHub, Gemini CLI, and 20+ other tools
- Solves a specific problem: packaging procedural knowledge and workflows
- Already provides progressive disclosure, multi-file structure, and validation
- **Not fragmented** - single standard adopted across ecosystem

**AI Resource Spec landscape:**
- Prompts and rules have **no unified standard**
- Every tool uses different formats for instructions and constraints
- Fragmentation prevents portability and reuse
- **This is the problem AI Resource Spec solves**

### Key Insight

The AI Resource Specification exists to **unify fragmented standards**. It provides a canonical format that can be compiled to tool-specific formats.

```
AI Resource Spec → Compile → Tool-specific formats
```

Agent Skills don't need this because they're already unified. Adding them would create unnecessary abstraction:

```
AI Resource Spec → Agent Skills → Already works everywhere
     ↑
  Unnecessary layer
```

## Decision

**Agent Skills are explicitly out of scope for the AI Resource Specification.**

The specification will focus on:
- Prompts (instructions)
- Rules (constraints)
- Collections (Promptsets, Rulesets)
- Fragment composition

Agent Skills remain a separate, complementary standard.

## Rationale

1. **Skills aren't fragmented** - They already have a successful, adopted standard
2. **Avoid unnecessary abstraction** - Wrapping a working standard adds no value
3. **Complementary, not overlapping** - Skills handle capabilities/workflows, AI Resource Spec handles instructions/constraints
4. **Different problems** - Skills need multi-file structure and progressive disclosure; prompts/rules need composition and compilation
5. **Respect existing success** - Agent Skills solved their problem well; don't reinvent it

## Consequences

### Positive

- Spec remains focused on solving actual fragmentation
- No impedance mismatch with Agent Skills directory structure
- Users can use both standards together naturally
- Simpler specification (4 resource kinds, not 6)
- Respects and acknowledges Agent Skills' success

### Negative

- Users must understand two separate standards
- No unified "AI artifact" abstraction
- Can't use AI Resource Spec features (fragments, validation) with skills

### Why the negatives are acceptable

- The standards serve different purposes
- Agent Skills already has validation and tooling
- Fragments don't make sense for multi-file skill structures
- Forcing unification would harm both standards

## Relationship Between Standards

**Agent Skills**: Procedural knowledge and workflows
- "How to process PDFs"
- "How to analyze datasets"
- Bundles scripts, references, assets

**AI Resource Spec**: Declarative instructions and constraints
- "Summarize this code"
- "Never hardcode secrets"
- Composition and reuse patterns

**Used together:**
```
project/
├── .ai-resources/
│   ├── prompts.yml      # AI Resource Spec
│   └── rules.yml        # AI Resource Spec
└── skills/
    ├── pdf-processing/  # Agent Skills
    └── data-analysis/   # Agent Skills
```

## Documentation Updates

- Add note to README.md explaining relationship with Agent Skills
- Remove research/skills/ directory (research complete)
- Update TASK.md to reflect this decision

## References

- Agent Skills specification: https://agentskills.io/specification
- Agent Skills adoption: https://agentskills.io/overview
- TASK.md: Original design exploration

## Notes

This decision was reached after thorough research into the Agent Skills format and ecosystem. The research confirmed that Skills have achieved what AI Resource Spec aims to achieve for prompts and rules: a unified, portable standard with wide adoption.

The right move is to acknowledge this success and keep the standards separate and complementary.

---

**Date**: 2026-02-19  
**Supersedes**: None  
**Superseded by**: None (current)
