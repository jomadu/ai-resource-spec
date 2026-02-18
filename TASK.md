# TASK: Refactor Fragments from Standalone Resources to Inline Definitions

## Overview

Remove `kind: Fragment` as a standalone resource type. Instead, fragments become inline definitions within Rule, Ruleset, Prompt, and Promptset resources.

---

## Current Model (To Be Removed)

### Standalone Fragment Resource
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
    focus:
      type: string
  body: "Read {{path}}{{#focus}} and focus on {{focus}}{{/focus}}"
```

### Usage in Other Resources
```yaml
# prompt.yml
apiVersion: ai-resource/v1
kind: Prompt
metadata:
  id: implement-feature
spec:
  body:
    - fragment: read-file
      inputs:
        path: AGENTS.md
        focus: build system
    - "Now implement the feature"
```

### Fragment Resolution
- Fragments are standalone files in the same directory
- Resolution searches current directory only (same-directory resolution)
- Fragment IDs must be unique within directory
- Cross-file references via `fragment: <id>`

---

## New Model (Target State)

### Inline Fragment Definitions

Fragments are defined within the resource that uses them:

```yaml
apiVersion: ai-resource/v1
kind: Prompt
metadata:
  id: implement-feature
spec:
  fragments:
    read-file:
      inputs:
        path:
          type: string
          required: true
        focus:
          type: string
      body: "Read {{path}}{{#focus}} and focus on {{focus}}{{/focus}}"
  
  body:
    - fragment: read-file
      inputs:
        path: AGENTS.md
        focus: build system
    - "Now implement the feature"
```

### Ruleset Example

```yaml
apiVersion: ai-resource/v1
kind: Ruleset
metadata:
  id: security-rules
spec:
  fragments:
    check-input:
      inputs:
        field:
          type: string
          required: true
      body: "Always validate {{field}} before processing"
    
    section-header:
      inputs:
        title:
          type: string
          required: true
      body: |
        ## {{title}}
        Follow these guidelines:
  
  rules:
    validate-username:
      priority: 100
      enforcement: must
      body:
        - fragment: section-header
          inputs:
            title: Username Validation
        - fragment: check-input
          inputs:
            field: username
    
    validate-email:
      priority: 100
      enforcement: must
      body:
        - fragment: section-header
          inputs:
            title: Email Validation
        - fragment: check-input
          inputs:
            field: email
```

### Promptset Example

```yaml
apiVersion: ai-resource/v1
kind: Promptset
metadata:
  id: code-review
spec:
  fragments:
    read-file:
      inputs:
        path:
          type: string
          required: true
        focus:
          type: string
      body: "Read {{path}}{{#focus}} and focus on {{focus}}{{/focus}}"
    
    coding-standards:
      body: |
        Follow these standards:
        - Write minimal code
        - Add tests only when requested
  
  prompts:
    implement:
      body:
        - fragment: read-file
          inputs:
            path: AGENTS.md
            focus: build system
        - fragment: coding-standards
        - "Now implement the feature"
    
    refactor:
      body:
        - fragment: read-file
          inputs:
            path: src/main.py
        - fragment: coding-standards
        - "Refactor the code"
```

---

## Key Changes

### 1. Remove Standalone Fragment Resource

**Delete:**
- `kind: Fragment` as a resource type
- Fragment as a standalone file
- Cross-file fragment resolution
- Directory-scoped fragment namespace

### 2. Add Inline Fragment Definitions

**Add to Prompt, Promptset, Rule, Ruleset:**
```yaml
spec:
  fragments:           # Optional field
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

### 3. Fragment Scope

**Old:** Directory-scoped (fragments in same directory accessible to all resources)

**New:** Resource-scoped (fragments only accessible within the resource that defines them)

### 4. Fragment Resolution

**Old:** Search directory for `kind: Fragment` files with matching `metadata.id`

**New:** Lookup within `spec.fragments` of the same resource

### 5. Validation Rules

**Remove:**
- Fragment IDs must be unique within directory
- Fragment references must resolve to Fragment file in same directory

**Add:**
- Fragment IDs must be unique within resource
- Fragment references must resolve within `spec.fragments` of same resource

**Keep:**
- Fragment input type validation (full JSON Schema support)
- Mustache template syntax validation
- No circular fragment references

---

## Input Type System (Unchanged)

Fragments continue to support full JSON Schema-style type definitions:

### Primitives
```yaml
inputs:
  name:
    type: string
  count:
    type: number
  enabled:
    type: boolean
```

### Arrays of Primitives
```yaml
inputs:
  files:
    type: array
    items:
      type: string
```

### Arrays of Objects
```yaml
inputs:
  tasks:
    type: array
    items:
      type: object
      properties:
        name:
          type: string
        priority:
          type: number
        completed:
          type: boolean
```

### Nested Objects
```yaml
inputs:
  user:
    type: object
    properties:
      name:
        type: string
      address:
        type: object
        properties:
          city:
            type: string
          zip:
            type: number
```

---

## Rationale

### Why Remove Standalone Fragments?

1. **Fragments are authoring tools, not distribution assets**
   - They help avoid duplication within a resource
   - They're not meant to be shared across packages
   - They're compiled away at installation time

2. **Cross-file fragment sharing is a design smell**
   - If multiple resources need the same fragment, they should either:
     - Be in the same resource (Ruleset/Promptset)
     - Copy the content (it's just text)
     - Be separate packages (if truly independent)

3. **Simplifies the spec**
   - Removes a resource kind (5 → 4)
   - Removes directory namespace complexity
   - Removes cross-file resolution algorithm
   - Self-contained resources (everything in one file)

4. **Aligns with real-world usage**
   - Most packages are flat structures
   - Subdirectories are for organizing legacy raw files
   - New schema-based packages should be simple and flat

### Benefits of Inline Fragments

✅ **Self-contained:** Everything in one file
✅ **DRY within resource:** Reuse fragments across rules/prompts in same collection
✅ **No namespace complexity:** Fragments scoped to resource
✅ **Mustache templating:** Parameterization, conditionals, loops
✅ **Simple resolution:** Just lookup within same resource
✅ **Simpler implementation:** No cross-file dependencies

### Tradeoffs

❌ **No cross-file reuse:** Can't share fragments between resources
❌ **Duplication across files:** Common patterns must be copied

**Acceptable because:**
- Most resources don't need shared fragments
- If you need shared patterns, copy them (it's just text)
- External tooling can handle complex generation if needed
- Keeps the spec simple and focused

---

## Areas to Update

### Version Labeling Correction

**Rename `v1` to `draft` throughout the repository:**

1. **Directory structure:**
   - `spec/v1/` → `spec/draft/`
   - `schema/v1/` → `schema/draft/`
   - `examples/v1/` → `examples/draft/`

2. **File references:**
   - Update all internal links in markdown files
   - Update schema `$id` fields: `https://ai-resource.dev/schema/v1/` → `https://ai-resource.dev/schema/draft/`
   - Update `apiVersion` in examples: `ai-resource/v1` → `ai-resource/draft`

3. **Documentation:**
   - README.md: Update version references
   - All spec documents: Update paths and version references

### Specification Documents

1. **spec/draft/resource-kinds.md** (renamed from v1)
   - Remove Fragment section
   - Add `spec.fragments` field to Prompt, Promptset, Rule, Ruleset
   - Update body field examples to show inline fragment usage

2. **spec/draft/fragments.md** (renamed from v1)
   - Rewrite to describe inline fragments instead of standalone
   - Update resolution algorithm (resource-scoped, not directory-scoped)
   - Update examples to show inline definitions

3. **spec/draft/envelope.md** (renamed from v1)
   - Update resource kind list (remove Fragment)

4. **spec/draft/validation.md** (renamed from v1)
   - Update validation rules for inline fragments
   - Remove directory-scoped fragment validation
   - Add resource-scoped fragment validation

5. **spec/draft/implementation.md** (renamed from v1)
   - Update fragment resolution algorithm
   - Remove directory loading/indexing
   - Update to resource-scoped lookup

### JSON Schemas

1. **schema/draft/Fragment.schema.json** (DELETE - was in v1/)
   - DELETE this file

2. **schema/draft/Prompt.schema.json** (renamed from v1/)
   - Add `spec.fragments` field (optional)
   - Define fragment definition schema

3. **schema/draft/Promptset.schema.json** (renamed from v1/)
   - Add `spec.fragments` field (optional)
   - Define fragment definition schema

4. **schema/draft/Rule.schema.json** (renamed from v1/)
   - Add `spec.fragments` field (optional)
   - Define fragment definition schema

5. **schema/draft/Ruleset.schema.json** (renamed from v1/)
   - Add `spec.fragments` field (optional)
   - Define fragment definition schema

### Examples

1. **examples/draft/basic/** (renamed from v1/)
   - Remove `fragment.yml`
   - Update examples to show inline fragments

2. **examples/draft/composition/** (renamed from v1/)
   - Rewrite to show inline fragment composition
   - Update `read-file.yml`, `coding-standards.yml`, `prompt.yml`, `promptset.yml`

3. **examples/draft/real-world/** (renamed from v1/)
   - Update to use inline fragments
   - Update `code-review.yml`, `analyze-files.yml`, etc.

### Tests

1. **schema/draft/tests/valid/** (renamed from v1/)
   - Remove `fragment-basic.yml`, `fragment-complex-inputs.yml`
   - Add examples with inline fragments

2. **schema/draft/tests/invalid/** (renamed from v1/)
   - Remove `fragment-invalid-id.yml`, `fragment-missing-body.yml`
   - Add invalid inline fragment examples

### ADRs

1. **ADRs/001-fragments-as-resources.md**
   - UPDATE or SUPERSEDE with new ADR
   - Document decision to move fragments inline
   - Explain rationale (authoring tools, not distribution assets)
   - Address concerns about cross-file reuse

2. **NEW ADR: Inline Fragments**
   - Document the inline fragment model
   - Explain scope (resource-scoped, not directory-scoped)
   - Rationale for simplification
   - Tradeoffs (no cross-file reuse vs. simplicity)

### README

1. **README.md**
   - Update resource kind list (remove Fragment)
   - Update examples to show inline fragments
   - Update quick start examples

---

## Success Criteria

- [ ] All `v1` directories renamed to `draft`
- [ ] All `apiVersion: ai-resource/v1` changed to `apiVersion: ai-resource/draft`
- [ ] All schema `$id` URLs updated to use `/draft/` instead of `/v1/`
- [ ] All internal documentation links updated
- [ ] `kind: Fragment` removed from spec
- [ ] Inline `spec.fragments` added to Prompt, Promptset, Rule, Ruleset
- [ ] All JSON schemas updated
- [ ] All examples updated to use inline fragments
- [ ] All tests updated
- [ ] ADR updated or new ADR written
- [ ] Implementation guide updated
- [ ] README updated

---

## Notes

- **Current status:** Spec is in DRAFT phase (v1 was mistakenly named v1 instead of draft)
- This refactoring can be done without breaking changes since we're still in draft
- **No existing users or production packages** - no migration guide needed
- Implementations are not yet stable
- This is the right time to make this simplification before v1 release
