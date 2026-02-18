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

**Latest stable:** [v1](spec/v1/)

## Quick Start

### Simple Prompt

```yaml
apiVersion: ai-resource/v1
kind: Prompt
metadata:
  id: summarize
spec:
  body: "Summarize the following code"
```

### Prompt with Fragment

```yaml
# read-file.yml
apiVersion: ai-resource/v1
kind: Fragment
metadata:
  id: read-file
spec:
  inputs:
    path:
      type: string
      required: true
  body: "Read {{path}}"
```

```yaml
# prompt.yml
apiVersion: ai-resource/v1
kind: Prompt
metadata:
  id: implement
spec:
  body:
    - fragment: read-file
      inputs:
        path: AGENTS.md
    - "Implement the feature"
```

### Rule

```yaml
apiVersion: ai-resource/v1
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

### Version 1 (Current)

**Core Specification:**
- **[envelope.md](spec/v1/envelope.md)** - Resource envelope structure and metadata
- **[resource-kinds.md](spec/v1/resource-kinds.md)** - All resource kinds and their specifications
- **[rules.md](spec/v1/rules.md)** - Rule-specific behavior (enforcement, scope, priority)
- **[fragments.md](spec/v1/fragments.md)** - Fragment composition and template syntax
- **[validation.md](spec/v1/validation.md)** - Validation requirements and conformance

**Implementation:**
- **[implementation.md](spec/v1/implementation.md)** - Implementation guide and algorithms

**Schemas:**
- [schema/v1/](schema/v1/) - JSON Schema definitions for all resource kinds

**Examples:**
- [examples/v1/](examples/v1/) - Example resources demonstrating usage

## Architecture Decisions

- **[ADRs/](ADRs/)** - Architecture Decision Records documenting key design choices

## Versioning

Schema versions are defined under `/schema/v<major>/`. Breaking changes require a new major version. Backward-compatible changes may be introduced within the same version.

## Implementations

All implementations must conform to this specification:

- **ai-resource-core-go** - Go implementation

## Contributing

This repository is the canonical source of truth for the AI Resource contract. Changes to the specification require careful consideration of backward compatibility and implementation impact.
