# Validation (draft)

All resources must be validated in two phases: schema validation followed by semantic validation.

## Validation Order

Implementations MUST validate in this order:

1. **Schema Validation** - Structural correctness against JSON Schema
2. **Semantic Validation** - Logical correctness beyond schema constraints

If schema validation fails, semantic validation SHOULD be skipped.

## Schema Files

JSON schemas are provided for each resource kind:

- `/schema/draft/Prompt.schema.json`
- `/schema/draft/Promptset.schema.json`
- `/schema/draft/Rule.schema.json`
- `/schema/draft/Ruleset.schema.json`

## Phase 1: Schema Validation

Schema validation ensures structural correctness. All resources must validate against their JSON Schema.

### Required Fields

All required fields must be present:

- `apiVersion` (all resources)
- `kind` (all resources)
- `metadata` (all resources)
- `metadata.id` (all resources)
- `spec` (all resources)
- `spec.body` (all resources)
- `spec.enforcement` (Rule, Ruleset)

**Example error:**
```
Error: Missing required field 'metadata.id' in Prompt resource
```

### Field Types

All fields must match their schema-defined types:

- `apiVersion`: string
- `kind`: string
- `metadata`: object
- `metadata.id`: string
- `metadata.name`: string (optional)
- `metadata.description`: string (optional)
- `spec`: object
- `spec.body`: string or array
- `spec.fragments`: object (optional)
- `spec.enforcement`: string (Rule, Ruleset)
- `spec.scope`: array (Rule, Ruleset, optional)
- `spec.priority`: number (Rule, Ruleset, optional)
- `spec.items`: array (Promptset, Ruleset)

**Example error:**
```
Error: Field 'spec.priority' expects type 'number', got 'string' in Rule 'no-secrets'
```

### Pattern Matching

String fields with patterns must match:

- `metadata.id`: `^[a-zA-Z0-9_-]+$`
- Fragment keys in `spec.fragments`: `^[a-zA-Z0-9_-]+$`
- Fragment input keys: `^[a-zA-Z0-9_-]+$`
- Item keys in `spec.items`: `^[a-zA-Z0-9_-]+$`

**Example error:**
```
Error: metadata.id 'my prompt' does not match pattern ^[a-zA-Z0-9_-]+$
```

### Enum Validation

Enum fields must have valid values:

- `apiVersion`: `ai-resource/draft`
- `kind`: `Prompt`, `Promptset`, `Rule`, `Ruleset`
- `spec.enforcement`: `may`, `should`, `must`
- Fragment input `type`: `string`, `number`, `boolean`

**Example error:**
```
Error: spec.enforcement 'required' is not valid. Must be one of: may, should, must
```

### Collection Constraints

Collections must meet minimum size requirements:

- `spec.items` (Promptset, Ruleset): minimum 1 item
- `spec.scope` (Rule, Ruleset): minimum 1 scope entry when present
- `spec.body` as array: minimum 1 element

**Example error:**
```
Error: spec.items must contain at least 1 item in Promptset 'common-prompts'
```

## Phase 2: Semantic Validation

Semantic validation ensures logical correctness beyond structural constraints.

### Fragment Reference Resolution

All fragment references in `spec.body` must exist in `spec.fragments`:

**Example error:**
```
Error: Fragment 'read-file' referenced in Prompt 'implement-feature' not found in spec.fragments
```

**Valid example:**
```yaml
spec:
  fragments:
    read-file:
      body: "Read {{path}}"
  body:
    - fragment: read-file  # ✓ Exists in spec.fragments
```

**Invalid example:**
```yaml
spec:
  fragments:
    read-file:
      body: "Read {{path}}"
  body:
    - fragment: write-file  # ✗ Not in spec.fragments
```

### Fragment Input Validation

#### Required Inputs Provided

All required fragment inputs must be provided when referencing a fragment:

**Example error:**
```
Error: Required input 'path' not provided for fragment 'read-file' in Prompt 'implement-feature'
```

**Valid example:**
```yaml
fragments:
  read-file:
    inputs:
      path:
        type: string
        required: true
body:
  - fragment: read-file
    inputs:
      path: "README.md"  # ✓ Required input provided
```

**Invalid example:**
```yaml
fragments:
  read-file:
    inputs:
      path:
        type: string
        required: true
body:
  - fragment: read-file  # ✗ Missing required input 'path'
```

#### No Undefined Inputs

Fragment references must not provide inputs not defined in the fragment:

**Example error:**
```
Error: Undefined input 'filename' provided for fragment 'read-file'. Valid inputs: path
```

**Valid example:**
```yaml
fragments:
  read-file:
    inputs:
      path:
        type: string
body:
  - fragment: read-file
    inputs:
      path: "README.md"  # ✓ Defined in fragment
```

**Invalid example:**
```yaml
fragments:
  read-file:
    inputs:
      path:
        type: string
body:
  - fragment: read-file
    inputs:
      filename: "README.md"  # ✗ Not defined in fragment
```

#### Input Type Matching

Provided inputs must match their defined types:

**Example error:**
```
Error: Input 'count' expects type 'number', got 'string' in fragment 'list-items'
```

**Valid example:**
```yaml
fragments:
  list-items:
    inputs:
      count:
        type: number
body:
  - fragment: list-items
    inputs:
      count: 5  # ✓ Correct type
```

**Invalid example:**
```yaml
fragments:
  list-items:
    inputs:
      count:
        type: number
body:
  - fragment: list-items
    inputs:
      count: "5"  # ✗ Wrong type (string instead of number)
```

### Body Format Validation

`spec.body` must be either a string or an array of valid body elements:

**Valid body elements:**
- String literals
- Fragment references (object with `fragment` key)

**Example error:**
```
Error: Invalid body element at index 1: expected string or fragment reference, got number
```

**Valid examples:**
```yaml
# String body
body: "Simple prompt text"

# Array body with strings
body:
  - "First part"
  - "Second part"

# Array body with fragment references
body:
  - fragment: read-file
    inputs:
      path: "README.md"
  - "Analyze the content"

# Mixed array body
body:
  - "Context:"
  - fragment: read-file
    inputs:
      path: "README.md"
  - "Now implement the feature"
```

**Invalid examples:**
```yaml
# Invalid: number in body array
body:
  - "Text"
  - 123  # ✗ Not a string or fragment reference

# Invalid: object without 'fragment' key
body:
  - "Text"
  - inputs:  # ✗ Missing 'fragment' key
      path: "README.md"
```

### Collection Key Uniqueness

Keys in collections must be unique:

- Fragment keys in `spec.fragments`
- Item keys in `spec.items`

**Example error:**
```
Error: Duplicate fragment key 'read-file' in Prompt 'implement-feature'
```

**Valid example:**
```yaml
fragments:
  read-file:
    body: "Read file"
  write-file:
    body: "Write file"
```

**Invalid example:**
```yaml
fragments:
  read-file:
    body: "Read file"
  read-file:  # ✗ Duplicate key
    body: "Read file again"
```

### Metadata ID Uniqueness

Within a single file, `metadata.id` must be unique across all resources:

**Example error:**
```
Error: Duplicate metadata.id 'summarize' found in file 'prompts.yml'
```

**Valid example:**
```yaml
---
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: summarize
spec:
  body: "Summarize"
---
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: implement  # ✓ Different ID
spec:
  body: "Implement"
```

**Invalid example:**
```yaml
---
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: summarize
spec:
  body: "Summarize"
---
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: summarize  # ✗ Duplicate ID
spec:
  body: "Summarize again"
```

## Error Message Requirements

All validation errors MUST include:

1. **Field path** - Dot-notation path to the invalid field (e.g., `spec.fragments.read-file.inputs.path`)
2. **Resource context** - Resource kind and ID (e.g., `in Prompt 'implement-feature'`)
3. **Clear description** - What is wrong and why

**Error message format:**
```
Error: <description> at <field-path> in <kind> '<id>'
```

**Examples:**
```
Error: Missing required field 'metadata.id' in Prompt resource
Error: Fragment 'read-file' referenced in Prompt 'implement-feature' not found in spec.fragments
Error: Input 'count' expects type 'number', got 'string' at spec.body[0].inputs.count in Prompt 'list-files'
```

## Conformance Testing

Implementations should validate against the test suite:

- Valid examples must pass both schema and semantic validation
- Invalid examples must fail validation with appropriate errors
- Fragment resolution must work correctly
- Input validation must enforce types and requirements

## Validation Timing

Implementations may validate at different times:

- **Load time:** Validate when loading resources from disk
- **Composition time:** Validate when resolving fragments
- **Runtime:** Validate when using resources

All validation must occur before resource use.

## Reference Implementation

The [ai-resource-core-go](https://github.com/maxdunn210/ai-resource-core-go) implementation provides a reference for validation behavior. See its test suite for comprehensive validation examples.
