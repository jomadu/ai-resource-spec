# ADR-001: Fragments as First-Class Resources

## Status

Superseded by [ADR-002](002-inline-fragments.md)

## Context

The AI Resource Specification defines a declarative format for AI artifacts (Prompts, Rules, and their collections). During design, a fundamental question emerged: **how should users handle reusable instruction patterns?**

### The Problem

Users writing multiple prompts or rules frequently need to reuse common patterns:
- "Read {{path}} and focus on {{focus}}" appears across multiple prompts
- Coding standards need to be referenced in multiple contexts
- File processing instructions are repeated with different parameters

Without a reuse mechanism, users face:
- **Duplication**: Copy-paste the same text across multiple resources
- **Maintenance burden**: Updates require changing multiple files
- **Inconsistency**: Duplicated text drifts over time
- **No parameterization**: Can't customize reused text per usage

### Proposed Solutions

Multiple approaches were considered:

#### 1. Fragments as Resources (Chosen)
```yaml
# Fragment definition
apiVersion: ai-resource/v1
kind: Fragment
metadata:
  id: read-file
spec:
  inputs:
    path: {type: string, required: true}
  body: "Read {{path}}"

# Fragment usage
apiVersion: ai-resource/v1
kind: Prompt
metadata:
  id: implement
spec:
  body:
    - fragment: read-file
      inputs: {path: AGENTS.md}
    - "Implement the feature"
```

**Pros**: Self-contained, validated, parameterized, directory-scoped
**Cons**: Adds a new resource kind, conceptually impure (meta-resource)

#### 2. Embedded Fragments
```yaml
apiVersion: ai-resource/v1
kind: Promptset
spec:
  fragments:
    read-file:
      inputs: {path: string}
      body: "Read {{path}}"
  prompts:
    implement:
      body:
        - fragment: read-file
          inputs: {path: AGENTS.md}
```

**Pros**: Scoped to collection, clear ownership
**Cons**: Can't share across collections, duplication across Promptsets/Rulesets

#### 3. External Templating
```bash
# Use Mustache/Jinja/etc with build step
mustache data.json prompts.template.yml > prompts.yml
```

**Pros**: Leverages existing tools, separation of concerns
**Cons**: Breaks portability, requires build tooling, not self-contained

#### 4. YAML Anchors
```yaml
.templates:
  read-file: &read-file "Read {{path}}"

spec:
  body:
    - *read-file
```

**Pros**: Native YAML feature, no new concepts
**Cons**: No parameterization, single-file only, no validation

#### 5. Simple Variable Substitution
```yaml
spec:
  variables: {file: AGENTS.md}
  body: "Read ${file}"
```

**Pros**: Simple, no template engine needed
**Cons**: No conditionals, no loops, limited expressiveness

### Key Tensions

**Purity vs. Pragmatism**: Fragments are meta-resources (resources about resources), which violates the "resources are artifacts" model. However, composition is a practical necessity.

**Simplicity vs. Power**: Adding Fragments increases spec complexity. However, the alternatives are either too limited or require external dependencies.

**Self-contained vs. External**: External templating tools are powerful but break the portability guarantee that a directory of resources should be complete and parseable.

### Precedents

**Terraform**: Uses `terraform_data` (formerly `null_resource`) as a meta-resource for lifecycle management and orchestration. While controversial, it demonstrates that meta-resources can work at scale when properly constrained.

**Kubernetes**: ConfigMaps and Secrets are not workloads but configuration resources that support workloads. Shows precedent for supporting resources in infrastructure specs.

**Docker**: Multi-stage builds use intermediate stages that aren't the final artifact. Demonstrates composition patterns in declarative systems.

## Decision

**We will include Fragment as a first-class resource kind in the specification.**

Fragments will:
- Use the standard resource envelope (apiVersion, kind, metadata, spec)
- Support typed inputs with JSON Schema-style definitions
- Use Mustache for template rendering
- Be scoped to their directory (flat namespace by metadata.id)
- Be prohibited from nesting (fragments cannot reference other fragments)

### Rationale

1. **The problem is real**: Analysis of examples shows 5:1 duplication reduction with Fragments. Composition is essential for systems that scale beyond toy examples.

2. **Alternatives are inadequate**:
   - External templating breaks portability
   - YAML anchors lack parameterization and cross-file support
   - Embedded fragments force duplication across collections
   - Simple interpolation lacks necessary expressiveness (conditionals, loops)

3. **Design is constrained**: Deliberate limitations prevent complexity creep:
   - No nesting (prevents dependency graphs)
   - Directory-scoped (prevents global namespace issues)
   - Single template language (Mustache is proven, widely supported)
   - No imports or package management

4. **Utility justifies conceptual cost**: While Fragments are meta-resources, they enable DRY principles while maintaining the self-contained, portable nature of resource directories.

5. **Proven pattern**: Terraform's evolution from `null_resource` to `terraform_data` shows meta-resources can be refined and work at scale.

## Consequences

### Positive

- **Eliminates duplication**: Users can define reusable templates once
- **Parameterization**: Templates can be customized per usage
- **Validation**: Fragment inputs are type-checked
- **Self-contained**: No external build tools required
- **Portable**: Directories remain complete and parseable
- **Familiar**: Mustache is a 15-year-old standard with libraries in every language

### Negative

- **Conceptual impurity**: Fragments are not end-user artifacts, they're composition mechanisms
- **Implementation burden**: All implementations must support Mustache rendering
- **Learning curve**: Users must understand fragments vs. prompts/rules
- **Potential for misuse**: Users might over-abstract or create overly complex templates

### Risks and Mitigations

**Risk**: Complexity creep—future requests for fragment nesting, imports, packages, etc.

**Mitigation**: Document explicit constraints. Future versions must NOT add:
- Fragment nesting
- Cross-directory imports  
- Multiple template languages
- Turing-complete templates

If these are needed, it indicates a design failure requiring fundamental rethink.

**Risk**: Users don't actually need Fragments (premature optimization)

**Mitigation**: Gather feedback from real-world usage. Be willing to deprecate in v2 if the feature proves unnecessary or if better alternatives emerge.

**Risk**: Fragments become a "junk drawer" for miscellaneous functionality

**Mitigation**: Clear documentation on when to use Fragments vs. simple strings. Fragments should only be used for genuinely reusable patterns, not as a general-purpose programming construct.

## Constraints

This decision is **provisional** and subject to the following conditions:

1. **Documentation must be explicit** about Fragment purpose, usage patterns, and anti-patterns
2. **Design must remain constrained**—no feature additions that increase complexity
3. **Spec must remain open** to alternative composition models if better approaches emerge

## References

- [fragments.md](fragments.md) - Fragment specification
- [resource-kinds.md](resource-kinds.md) - All resource kinds
- [implementation.md](implementation.md) - Implementation guide including fragment resolution
- Terraform `terraform_data` documentation
- Mustache specification: https://mustache.github.io/

## Notes

This ADR documents a decision made during initial specification design. The debate revealed fundamental tensions between purity and pragmatism, simplicity and power, self-contained and external approaches.

The decision favors pragmatism, constrained power, and self-contained design. However, the concerns raised by proponents of purity and simplicity are valid and must be continuously considered as the specification evolves.

Future ADRs should reference this decision when considering:
- Additional composition mechanisms
- Changes to Fragment capabilities
- Alternative reuse patterns
- Deprecation or replacement of Fragments

---

**Date**: 2026-02-17  
**Participants**: Specification design team  
**Supersedes**: None  
**Superseded by**: [ADR-002](002-inline-fragments.md) (2026-02-18)
