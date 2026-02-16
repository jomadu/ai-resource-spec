# AI Resource Specification

The AI Resource Specification defines a versioned, declarative format for describing portable AI artifacts such as prompts, rules, skills, tools, evaluations, and other structured AI assets.

This repository is the canonical source of truth for the AI Resource contract. It defines:

- The resource envelope (apiVersion, kind, metadata, spec)
- JSON Schema for structural validation
- Versioning and compatibility guarantees
- Normative behavioral rules

The specification is language-agnostic and contains no runtime or implementation code.

## Goals

- Provide a stable, extensible resource model
- Enable deterministic validation across implementations
- Support multiple resource kinds (singular and collections)
- Allow forward-compatible evolution

## Resource Envelope

All resources follow a Kubernetes-style envelope structure:

```yaml
apiVersion: ai-resource/v1    # Required: API version
kind: <ResourceKind>          # Required: Resource type
metadata:                     # Required: Resource metadata
  id: <identifier>            # Required: Machine identifier
  name: <human-name>          # Optional: Human-readable name
  description: <text>         # Optional: Description
spec:                         # Required: Resource-specific content
  # ... kind-specific fields
```

## Resource Kinds

### Prompt (Singular)

A single AI prompt resource.

```yaml
apiVersion: ai-resource/v1
kind: Prompt
metadata:
  id: summarize
  name: Summarize Code
  description: Summarizes code for review
spec:
  body: |
    Summarize the following code...
```

### Promptset (Collection)

A collection of related prompts.

```yaml
apiVersion: ai-resource/v1
kind: Promptset
metadata:
  id: code-review
  name: Code Review Prompts
  description: Prompts for code review workflows
spec:
  prompts:
    summarize:
      name: Summarize Code
      description: Summarizes code for review
      body: |
        Summarize the following code...
    explain:
      name: Explain Logic
      body: |
        Explain the logic of...
```

### Rule (Singular)

A single AI rule resource with enforcement and scope.

```yaml
apiVersion: ai-resource/v1
kind: Rule
metadata:
  id: no-magic-numbers
  name: No Magic Numbers
  description: Avoid magic numbers in code
spec:
  enforcement: must           # may | should | must
  scope:
    - files: ["**/*.py"]
  body: |
    Avoid using magic numbers...
```

### Ruleset (Collection)

A collection of related rules.

```yaml
apiVersion: ai-resource/v1
kind: Ruleset
metadata:
  id: clean-code
  name: Clean Code Rules
  description: Rules for writing clean code
spec:
  rules:
    no-magic-numbers:
      name: No Magic Numbers
      description: Avoid magic numbers
      priority: 100           # Optional: default 100
      enforcement: must       # may | should | must
      scope:
        - files: ["**/*.py"]
      body: |
        Avoid using magic numbers...
    descriptive-names:
      name: Use Descriptive Names
      enforcement: should
      body: |
        Use descriptive variable names...
```

## Metadata Fields

All resources require a `metadata` section with:

- `id` (required): Machine identifier, must match `^[a-zA-Z0-9_-]+$`
- `name` (optional): Human-readable name
- `description` (optional): Resource description

## Rule-Specific Fields

### Enforcement Levels

Rules support three enforcement levels:

- `may`: Optional suggestion
- `should`: Recommended practice
- `must`: Required constraint

### Scope

Rules can target specific files using glob patterns:

```yaml
scope:
  - files: ["**/*.py"]
  - files: ["src/**/*.ts", "lib/**/*.ts"]
```

## Collection Structure

Collections (Promptset, Ruleset) use maps (not arrays) for items:

- Keys must match `^[a-zA-Z0-9_-]+$`
- Keys must be unique within the collection
- Minimum one item required

## Versioning

Schema versions are defined under `/schema/<version>/`. Breaking changes require a new major version. Backward-compatible changes may be introduced within the same version.

Current version: `ai-resource/v1`

## Validation

JSON schemas are provided for each resource kind:

- `/schema/ai-resource-v1/Prompt.schema.json`
- `/schema/ai-resource-v1/Promptset.schema.json`
- `/schema/ai-resource-v1/Rule.schema.json`
- `/schema/ai-resource-v1/Ruleset.schema.json`

All implementations must validate resources against these schemas.

## Implementations

All implementations, including `ai-resource-core-go` and `ai-resource-manager`, must conform to this specification.