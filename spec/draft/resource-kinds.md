# Resource Kinds (draft)

The specification defines four resource kinds: two singular types and two collection types.

## Prompt

A single prompt resource.

### Structure

```yaml
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: <id>
  name: <name>           # Optional
  description: <text>    # Optional
spec:
  fragments:             # Optional: inline fragment definitions
    <fragment-id>:
      inputs:            # Optional: fragment parameters
        <name>:
          type: <type>
          required: <bool>
          default: <value>
      body: <template>   # Mustache template string
  body: <body>           # string | string[] | (string | FragmentRef)[]
```

### Example

```yaml
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: summarize
  name: Summarize Code
  description: Summarizes code for review
spec:
  body: "Summarize the following code for review"
```

### Example with Inline Fragment

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
        focus:
          type: string
      body: "Read {{path}}{{#focus}} and focus on {{focus}}{{/focus}}"
  body:
    - fragment: read-file
      inputs:
        path: AGENTS.md
        focus: build system
    - "Implement the feature"
```

## Promptset

A collection of related prompts.

### Structure

```yaml
apiVersion: ai-resource/draft
kind: Promptset
metadata:
  id: <id>
  name: <name>           # Optional
  description: <text>    # Optional
spec:
  fragments:             # Optional: inline fragment definitions
    <fragment-id>:
      inputs:            # Optional: fragment parameters
        <name>:
          type: <type>
          required: <bool>
          default: <value>
      body: <template>   # Mustache template string
  prompts:
    <key>:               # Map key, pattern: ^[a-zA-Z0-9_-]+$
      name: <name>       # Optional
      description: <text> # Optional
      body: <body>       # string | string[] | (string | FragmentRef)[]
```

### Example

```yaml
apiVersion: ai-resource/draft
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

### Example with Inline Fragments

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
        - fragment: coding-standards
        - "Implement the feature"
    refactor:
      body:
        - fragment: coding-standards
        - "Refactor the code"
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
apiVersion: ai-resource/draft
kind: Rule
metadata:
  id: <id>
  name: <name>           # Optional
  description: <text>    # Optional
spec:
  fragments:             # Optional: inline fragment definitions
    <fragment-id>:
      inputs:            # Optional: fragment parameters
        <name>:
          type: <type>
          required: <bool>
          default: <value>
      body: <template>   # Mustache template string
  enforcement: <level>   # may | should | must
  scope:                 # Optional
    - files: [<patterns>]
  body: <body>           # string | string[] | (string | FragmentRef)[]
```

### Example

```yaml
apiVersion: ai-resource/draft
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
apiVersion: ai-resource/draft
kind: Ruleset
metadata:
  id: <id>
  name: <name>           # Optional
  description: <text>    # Optional
spec:
  fragments:             # Optional: inline fragment definitions
    <fragment-id>:
      inputs:            # Optional: fragment parameters
        <name>:
          type: <type>
          required: <bool>
          default: <value>
      body: <template>   # Mustache template string
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
apiVersion: ai-resource/draft
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

Fragment references resolve to inline fragments defined in the same resource's `spec.fragments` field. See [fragments.md](fragments.md) for complete details on fragment composition.
