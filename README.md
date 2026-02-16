# AI Resource Specification

The AI Resource Specification defines a versioned, declarative format for describing portable AI artifacts such as prompts, skills, tools, evaluations, and other structured AI assets.

This repository is the canonical source of truth for the AI Resource contract. It defines:

- The resource envelope (apiVersion, kind, metadata, spec)
- JSON Schema for structural validation
- Versioning and compatibility guarantees
- Normative behavioral rules

The specification is language-agnostic and contains no runtime or implementation code.

## Goals

- Provide a stable, extensible resource model
- Enable deterministic validation across implementations
- Support multiple resource kinds
- Allow forward-compatible evolution

## Resource Envelope

```yaml
apiVersion: ai-resource/v1
kind: Prompt
metadata:
  name: summarize
spec:
  template: "Summarize the following..."
```

All implementations, including `ai-resource-core-go` and `ai-resource-manager`, must conform to this specification.

## Versioning

Schema versions are defined under `/schema/<version>/`. Breaking changes require a new major version. Backward-compatible changes may be introduced within the same version.

This repository defines the contract. Implementations enforce it.