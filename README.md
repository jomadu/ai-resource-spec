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

## Goals

- Provide a stable, extensible resource model
- Enable deterministic validation across implementations
- Support composition through reusable fragments
- Allow forward-compatible evolution

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

### Core Specification

- **[ENVELOPE.md](ENVELOPE.md)** - Resource envelope structure and metadata
- **[RESOURCE-KINDS.md](RESOURCE-KINDS.md)** - All resource kinds and their specifications
- **[RULES.md](RULES.md)** - Rule-specific behavior (enforcement, scope, priority)
- **[FRAGMENTS.md](FRAGMENTS.md)** - Fragment composition and template syntax
- **[VALIDATION.md](VALIDATION.md)** - Validation requirements and conformance

### Implementation

- **[IMPLEMENTATION.md](IMPLEMENTATION.md)** - Implementation guide and algorithms

## Versioning

Current version: **ai-resource/v1**

Schema versions are defined under `/schema/<version>/`. Breaking changes require a new major version. Backward-compatible changes may be introduced within the same version.

## Implementations

All implementations must conform to this specification:

- **ai-resource-core-go** - Go implementation

## Contributing

This repository is the canonical source of truth for the AI Resource contract. Changes to the specification require careful consideration of backward compatibility and implementation impact.
