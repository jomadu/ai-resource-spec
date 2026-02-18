# Resource Kinds (v1)

The specification defines five resource kinds: two singular types, two collection types, and one template type.

## Prompt

A single prompt resource.

### Structure

```yaml
apiVersion: ai-resource/v1
kind: Prompt
metadata:
  id: <id>
  name: <name>           # Optional
  description: <text>    # Optional
spec:
  body: <body>           # string | string[] | (string | FragmentRef)[]
```

### Example

```yaml
apiVersion: ai-resource/v1
kind: Prompt
metadata:
  id: summarize
  name: Summarize Code
  description: Summarizes code for review
spec:
  body: "Summarize the following code for review"
```

## Promptset

A collection of related prompts.

### Structure

```yaml
apiVersion: ai-resource/v1
kind: Promptset
metadata:
  id: <id>
  name: <name>           # Optional
  description: <text>    # Optional
spec:
  prompts:
    <key>:               # Map key, pattern: ^[a-zA-Z0-9_-]+$
      name: <name>       # Optional
      description: <text> # Optional
      body: <body>       # string | string[] | (string | FragmentRef)[]
```

### Example

```yaml
apiVersion: ai-resource/v1
kind: Promptset
metadata:
  id: code-review
  name: Code Review Prompts
spec:
  prompts:
    summarize:
      name: Summarize Code
      body: "Summarize the code"
    explain:
      name: Explain Logic
      body: "Explain the logic"
```

### Collection Rules

- `prompts` is a map (not an array)
- Map keys must match pattern `^[a-zA-Z0-9_-]+$`
- Map keys must be unique within the collection
- Minimum one prompt required

## Rule

A single rule resource with enforcement level and optional scope.

### Structure

```yaml
apiVersion: ai-resource/v1
kind: Rule
metadata:
  id: <id>
  name: <name>           # Optional
  description: <text>    # Optional
spec:
  enforcement: <level>   # may | should | must
  scope:                 # Optional
    - files: [<patterns>]
  body: <body>           # string | string[] | (string | FragmentRef)[]
```

### Example

```yaml
apiVersion: ai-resource/v1
kind: Rule
metadata:
  id: no-magic-numbers
  name: No Magic Numbers
spec:
  enforcement: must
  scope:
    - files: ["**/*.py"]
  body: "Avoid using magic numbers in code"
```

## Ruleset

A collection of related rules.

### Structure

```yaml
apiVersion: ai-resource/v1
kind: Ruleset
metadata:
  id: <id>
  name: <name>           # Optional
  description: <text>    # Optional
spec:
  rules:
    <key>:               # Map key, pattern: ^[a-zA-Z0-9_-]+$
      name: <name>       # Optional
      description: <text> # Optional
      priority: <number> # Optional, default: 100
      enforcement: <level> # may | should | must
      scope:             # Optional
        - files: [<patterns>]
      body: <body>       # string | string[] | (string | FragmentRef)[]
```

### Example

```yaml
apiVersion: ai-resource/v1
kind: Ruleset
metadata:
  id: clean-code
  name: Clean Code Rules
spec:
  rules:
    no-magic-numbers:
      name: No Magic Numbers
      priority: 100
      enforcement: must
      scope:
        - files: ["**/*.py"]
      body: "Avoid magic numbers"
    descriptive-names:
      enforcement: should
      body: "Use descriptive names"
```

### Collection Rules

- `rules` is a map (not an array)
- Map keys must match pattern `^[a-zA-Z0-9_-]+$`
- Map keys must be unique within the collection
- Minimum one rule required

## Fragment

A reusable, parameterizable template.

### Structure

```yaml
apiVersion: ai-resource/v1
kind: Fragment
metadata:
  id: <id>
  name: <name>           # Optional
  description: <text>    # Optional
spec:
  inputs:                # Optional
    <key>:
      type: <type>       # string | number | boolean | array | object
      required: <bool>   # Optional, default: false
      default: <value>   # Optional
      items: <schema>    # For array type
      properties: <schema> # For object type
  body: <template>       # Mustache template string
```

### Example

```yaml
apiVersion: ai-resource/v1
kind: Fragment
metadata:
  id: read-file
  name: Read File Instruction
spec:
  inputs:
    path:
      type: string
      required: true
    focus:
      type: string
      default: ""
  body: |
    Read {{path}}{{#focus}} and focus on {{focus}}{{/focus}}.
```

## Body Field

The `body` field appears in Prompt, Rule, and collection items. It supports three formats:

### Simple String

```yaml
body: "Single instruction"
```

### Array of Strings

```yaml
body:
  - "First instruction"
  - "Second instruction"
```

### Array with Fragment References

```yaml
body:
  - fragment: read-file
    inputs:
      path: AGENTS.md
  - "Additional instructions"
```

See [FRAGMENTS.md](fragments.md) for complete details on fragment composition.
