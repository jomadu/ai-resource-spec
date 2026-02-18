# Fragment Composition (draft)

Fragments are reusable, parameterizable templates defined inline within resources.

## Overview

Fragments are defined in the `spec.fragments` field of Prompt, Promptset, Rule, and Ruleset resources. They are referenced by key and resolved at composition time.

```yaml
apiVersion: ai-resource/draft
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

## Fragment Definitions

Fragments are defined in the `spec.fragments` field as a map:

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

Fragment IDs must match pattern `^[a-zA-Z0-9_-]+$` and be unique within the resource.

## Body Formats

The `body` field supports three formats:

**Simple string:**
```yaml
body: "Single instruction"
```

**Array of strings:**
```yaml
body:
  - "First instruction"
  - "Second instruction"
```

**Array with fragment references:**
```yaml
body:
  - fragment: read-file
    inputs:
      path: AGENTS.md
  - "Additional instructions"
```

## Template Syntax

Fragments use Mustache template syntax.

**Substitution:**
```yaml
body: "Read {{path}}"
# path="AGENTS.md" → "Read AGENTS.md"
```

**Conditional:**
```yaml
body: "Read {{path}}{{#focus}} and focus on {{focus}}{{/focus}}"
# path="AGENTS.md", focus="tests" → "Read AGENTS.md and focus on tests"
# path="AGENTS.md", focus="" → "Read AGENTS.md"
```

**Array (primitives):**
```yaml
body: |
  Files:
  {{#files}}
  - {{.}}
  {{/files}}
# files=["main.py", "utils.py"] → "Files:\n- main.py\n- utils.py"
```

**Array (objects):**
```yaml
body: |
  {{#tasks}}
  - {{name}}: {{description}}
  {{/tasks}}
# tasks=[{name: "Build", description: "Compile"}] → "- Build: Compile"
```

## Input Types

Fragment inputs support JSON Schema-style type definitions.

### Structure

```yaml
inputs:
  <name>:
    type: string|number|boolean|array|object
    required: true|false    # Default: false
    default: <value>        # Optional
    items: <schema>         # For array type
    properties: <schema>    # For object type
```

### Primitives

```yaml
inputs:
  path:
    type: string
    required: true
  count:
    type: number
    default: 0
  enabled:
    type: boolean
    default: false
```

### Array of Primitives

```yaml
inputs:
  files:
    type: array
    items:
      type: string
    required: true
```

### Array of Objects

```yaml
inputs:
  tasks:
    type: array
    items:
      type: object
      properties:
        name:
          type: string
        description:
          type: string
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

## Resolution

Fragments are resolved by key from the same resource's `spec.fragments` field.

### Scope

- Fragments are scoped to the resource that defines them
- Fragment references resolve within `spec.fragments` of the same resource
- No cross-resource fragment sharing

### Process

**Input:**
```yaml
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
    - "Review the code"
```

**Steps:**
1. Resolve `read-file` fragment with inputs → "Read AGENTS.md"
2. Keep "Review the code" as-is → "Review the code"
3. Join with `\n\n` separator

**Output:**
```
Read AGENTS.md

Review the code
```

## Complete Example

**Promptset with inline fragments:**

```yaml
apiVersion: ai-resource/draft
kind: Promptset
metadata:
  id: dev-workflow
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

**Ruleset with inline fragments:**

```yaml
apiVersion: ai-resource/draft
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

## Design Decisions

**Inline definitions:** Fragments are defined within the resource that uses them, not as standalone files. This makes resources self-contained.

**Resource-scoped:** Fragments are only accessible within the resource that defines them. No cross-resource sharing.

**No nesting:** Fragments cannot reference other fragments. Leaf-level templates only.

**Mustache templates:** Standard template syntax with substitution, conditionals, and loops.
