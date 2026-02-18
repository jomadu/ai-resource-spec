# Fragment Composition (v1)

Fragments are reusable, parameterizable templates that compose into prompts and rules.

## Overview

Fragments are standalone resources that live in the same directory as the resources that use them. They are referenced by `metadata.id` and resolved at composition time.

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
      default: ""
  body: "Read {{path}}{{#focus}} and focus on {{focus}}{{/focus}}."
```

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

Fragments are resolved by `metadata.id` from resources in the same directory.

### Discovery

- All resources in a directory share a flat namespace
- Fragment references resolve by `metadata.id`
- Resolution fails if fragment ID is not found

### Process

**Input:**
```yaml
body:
  - fragment: read-file
    inputs:
      path: AGENTS.md
      focus: build system
  - "Review the code"
```

**Steps:**
1. Resolve `read-file` fragment with inputs → string
2. Keep "Review the code" as-is → string
3. Join with `\n\n` separator

**Output:**
```
Read AGENTS.md and focus on build system.

Review the code
```

## Complete Example

**Directory:**
```
project/
  read-file.yml
  coding-standards.yml
  prompts.yml
```

**read-file.yml:**
```yaml
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
  body: "Read {{path}}{{#focus}} and focus on {{focus}}{{/focus}}."
```

**coding-standards.yml:**
```yaml
apiVersion: ai-resource/v1
kind: Fragment
metadata:
  id: coding-standards
spec:
  body: |
    Follow these standards:
    - Write minimal code
    - Add tests only when requested
```

**prompts.yml:**
```yaml
apiVersion: ai-resource/v1
kind: Promptset
metadata:
  id: dev-prompts
spec:
  prompts:
    implement-feature:
      body:
        - fragment: read-file
          inputs:
            path: AGENTS.md
            focus: build system
        - fragment: coding-standards
        - "Now implement the feature"
    
    simple-prompt:
      body: "Just a simple prompt"
```

## Design Decisions

**Fragments are resources:** First-class resources with standard envelope structure. Live in separate files, share directory namespace.

**No special fields:** Collections have no `spec.fragments` or `spec.imports` fields. Fragment references resolve against directory namespace.

**No nesting:** Fragments cannot reference other fragments. Leaf-level templates only.

**Directory scope:** Fragments scoped to their directory. Flat namespace indexed by `metadata.id`.
