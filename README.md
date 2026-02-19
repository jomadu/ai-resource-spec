# AI Resource Specification

A versioned, declarative format for describing portable AI artifacts such as prompts, rules, and reusable templates.

## Overview

This specification defines:

- Resource envelope structure (apiVersion, kind, metadata, spec)
- Four resource kinds (Prompt, Promptset, Rule, Ruleset)
- Inline fragment composition for reusable templates
- JSON Schema for structural validation
- Versioning and compatibility guarantees

The specification is language-agnostic and contains no runtime or implementation code.

## Current Version

**Latest draft:** [draft](spec/draft/)

## Quick Start

### Simple Prompt

```yaml
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: summarize
spec:
  body: "Summarize the following code"
```

### Prompt with Inline Fragment

```yaml
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: implement
spec:
  fragments:
    read-file:
      inputs:
        path:
          type: string
          required: true
      body: "Read {{path}}"
  body:
    - fragment: read-file
      inputs:
        path: AGENTS.md
    - "Implement the feature"
```

### Rule

```yaml
apiVersion: ai-resource/draft
kind: Rule
metadata:
  id: no-secrets
spec:
  enforcement: must
  scope:
    - files: ["**/*.py"]
  body: "Never hardcode secrets in code"
```

## Documentation

### Draft Version (Current)

**Core Specification:**
- **[envelope.md](spec/draft/envelope.md)** - Resource envelope structure and metadata
- **[resource-kinds.md](spec/draft/resource-kinds.md)** - All resource kinds and their specifications
- **[rules.md](spec/draft/rules.md)** - Rule-specific behavior (enforcement, scope, priority)
- **[fragments.md](spec/draft/fragments.md)** - Fragment composition and template syntax
- **[validation.md](spec/draft/validation.md)** - Validation requirements and conformance

**Implementation:**
- **[implementation.md](spec/draft/implementation.md)** - Implementation guide and algorithms

**Schemas:**
- [schema/draft/](schema/draft/) - JSON Schema definitions for all resource kinds

**Examples:**
- [examples/draft/](examples/draft/) - Example resources demonstrating usage

## Architecture Decisions

- **[ADRs/](ADRs/)** - Architecture Decision Records documenting key design choices
  - [ADR-001](ADRs/001-fragments-as-resources.md) - Fragments as first-class resources (superseded)
  - [ADR-002](ADRs/002-inline-fragments.md) - Inline fragments instead of standalone resources
  - [ADR-003](ADRs/003-skills-out-of-scope.md) - Why Agent Skills are out of scope

## Versioning

Schema versions are defined under `/schema/v<major>/`. Breaking changes require a new major version. Backward-compatible changes may be introduced within the same version.

The current specification is in **draft** status and has not yet reached v1.

## Relationship to Agent Skills

The AI Resource Specification is complementary to [Agent Skills](https://agentskills.io), not overlapping:

- **Agent Skills**: Procedural knowledge and workflows (e.g., "how to process PDFs")
- **AI Resource Spec**: Declarative instructions and constraints (e.g., "summarize code", "never hardcode secrets")

Agent Skills already provides a unified, widely-adopted standard for packaging agent capabilities. The AI Resource Spec focuses on unifying fragmented prompt and rule formats across tools. See [ADR-003](ADRs/003-skills-out-of-scope.md) for details.

## Implementations

All implementations must conform to this specification:

- **ai-resource-core-go** - Go implementation

## Contributing

This repository is the canonical source of truth for the AI Resource contract. Changes to the specification require careful consideration of backward compatibility and implementation impact.
