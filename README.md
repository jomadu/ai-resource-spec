# AI Resource Specification

A versioned, declarative format for describing portable AI artifacts such as prompts, rules, and reusable templates.

## Overview

This specification defines:

- Resource envelope structure (apiVersion, kind, metadata, spec)
- Five resource kinds (Prompt, Promptset, Rule, Ruleset, Fragment)
- Fragment composition model for reusable templates
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

## Versioning

Schema versions are defined under `/schema/v<major>/`. Breaking changes require a new major version. Backward-compatible changes may be introduced within the same version.

The current specification is in **draft** status and has not yet reached v1.

## Implementations

All implementations must conform to this specification:

- **ai-resource-core-go** - Go implementation

## Contributing

This repository is the canonical source of truth for the AI Resource contract. Changes to the specification require careful consideration of backward compatibility and implementation impact.
