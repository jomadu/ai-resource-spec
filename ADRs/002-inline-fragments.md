# ADR-002: Inline Fragments Instead of Standalone Resources

## Status

Accepted

## Context

ADR-001 established Fragments as first-class resources (standalone files with `kind: Fragment`). After initial implementation and usage analysis, we identified several issues with this approach:

### Problems with Standalone Fragments

1. **Fragments are authoring tools, not distribution assets**
   - They help avoid duplication within a resource
   - They're compiled away at installation time
   - They're not meant to be shared across packages

2. **Cross-file fragment sharing is a design smell**
   - If multiple resources need the same fragment, they should either:
     - Be in the same resource (Ruleset/Promptset)
     - Copy the content (it's just text)
     - Be separate packages (if truly independent)

3. **Unnecessary complexity**
   - Directory namespace management
   - Cross-file resolution algorithm
   - Fragment ID uniqueness across directory
   - Resources split across multiple files

4. **Misaligned with real-world usage**
   - Most packages are flat structures
   - Fragments are rarely shared across resources
   - Self-contained resources are easier to understand and maintain

### Proposed Solution

Move fragments from standalone resources to inline definitions within the resources that use them.

**Before (Standalone):**
```yaml
# read-file.yml
apiVersion: ai-resource/draft
kind: Fragment
metadata:
  id: read-file
spec:
  inputs:
    path: {type: string, required: true}
  body: "Read {{path}}"
```

```yaml
# prompt.yml
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: implement
spec:
  body:
    - fragment: read-file
      inputs: {path: AGENTS.md}
```

**After (Inline):**
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
```

## Decision

**We will remove Fragment as a standalone resource kind and add inline fragment definitions to Prompt, Promptset, Rule, and Ruleset resources.**

### Changes

1. **Remove `kind: Fragment`** as a resource type
2. **Add `spec.fragments` field** to Prompt, Promptset, Rule, Ruleset
3. **Change scope**: Directory-scoped → Resource-scoped
4. **Update resolution**: Lookup within same resource's `spec.fragments`

### Fragment Definition Structure

```yaml
spec:
  fragments:
    <fragment-id>:
      inputs:          # Optional: fragment parameters
        <name>:
          type: <type>
          required: <bool>
          default: <value>
          items: <schema>      # For array type
          properties: <schema> # For object type
      body: <template>         # Mustache template string
```

## Rationale

### Benefits

1. **Self-contained resources**: Everything in one file, no cross-file dependencies
2. **Simpler specification**: 4 resource kinds instead of 5
3. **Simpler implementation**: No directory loading, no cross-file resolution
4. **Clearer scope**: Fragments only accessible within defining resource
5. **Better DRY within collections**: Rulesets/Promptsets can share fragments across items
6. **Easier to understand**: All context in one place

### Tradeoffs

**Lost capability**: Can't share fragments across resources in the same directory

**Why acceptable**:
- Most resources don't need shared fragments
- If you need shared patterns, copy them (it's just text)
- If patterns are truly reusable across packages, create a library
- Keeps the spec simple and focused on core use cases

### Comparison to Alternatives

**Embedded in collections only**: Would force duplication across Prompt/Rule resources

**External templating**: Breaks portability, requires build tooling

**Keep both models**: Unnecessary complexity, confusing for users

## Consequences

### Positive

- Resources are self-contained and portable
- No directory namespace complexity
- Simpler implementation (no cross-file resolution)
- Fragments scoped to where they're used
- Still supports DRY within Rulesets/Promptsets

### Negative

- Can't share fragments across resources
- Duplication if same fragment needed in multiple resources
- Migration required for existing standalone fragments

### Migration Path

Since the spec is still in draft (mislabeled as v1), no migration guide is needed. This change happens before v1 release.

## Implementation Impact

### Specification Changes

- Remove Fragment from envelope.md kind list
- Rewrite fragments.md for inline model
- Update resource-kinds.md with `spec.fragments` field
- Update validation.md for resource-scoped validation
- Update implementation.md for inline resolution

### Schema Changes

- Delete Fragment.schema.json
- Add `spec.fragments` to Prompt, Promptset, Rule, Ruleset schemas
- Add fragment definition schema to each

### Example Changes

- Delete standalone fragment files
- Update examples to use inline fragments
- Update test suite

## References

- [ADR-001](001-fragments-as-resources.md) - Original decision for standalone fragments
- [fragments.md](../spec/draft/fragments.md) - Updated fragment specification
- [resource-kinds.md](../spec/draft/resource-kinds.md) - Updated resource kinds

## Notes

This decision supersedes ADR-001. The original rationale for Fragments (avoiding duplication, parameterization, validation) remains valid. The change is in **scope** (resource-scoped vs directory-scoped) and **definition location** (inline vs standalone).

The core fragment capabilities remain unchanged:
- Mustache templating
- Full JSON Schema input types
- Type validation
- No nesting

---

**Date**: 2026-02-18  
**Supersedes**: ADR-001  
**Superseded by**: None (current)
