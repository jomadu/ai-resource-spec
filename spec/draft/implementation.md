# Implementation Guide (draft)

Reference guide for implementing the AI Resource Specification.

## Overview

Implementations must:
1. Load and parse resources from YAML/JSON
2. Validate against JSON Schema
3. Resolve inline fragment references
4. Render Mustache templates
5. Handle errors appropriately

## Resource Loading

### File Loading

Load resources from YAML/JSON files:

```
1. Parse file as YAML/JSON
2. Validate against schema for resource kind
3. Return resource object
```

### Multi-Document YAML

Support YAML files with multiple documents (separated by `---`):

```yaml
---
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: prompt-1
spec:
  body: "First prompt"
---
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: prompt-2
spec:
  body: "Second prompt"
```

## Fragment Resolution

### Algorithm

```
function resolveBody(body, fragments):
  if body is string:
    return body
  
  if body is array:
    resolved = []
    for item in body:
      if item is string:
        resolved.append(item)
      else if item is fragmentRef:
        fragment = fragments[item.fragment]
        if not fragment:
          error("Fragment not found: " + item.fragment)
        
        validateInputs(fragment.inputs, item.inputs)
        rendered = renderTemplate(fragment.body, item.inputs)
        resolved.append(rendered)
    
    return join(resolved, "\n\n")
```

### Input Validation

```
function validateInputs(inputDefs, providedInputs):
  for name, def in inputDefs:
    if def.required and name not in providedInputs:
      error("Required input missing: " + name)
    
    if name in providedInputs:
      validateType(def.type, providedInputs[name])
  
  for name in providedInputs:
    if name not in inputDefs:
      error("Unknown input: " + name)
```

### Type Validation

```
function validateType(typeDef, value):
  if typeDef.type == "string":
    assert value is string
  else if typeDef.type == "number":
    assert value is number
  else if typeDef.type == "boolean":
    assert value is boolean
  else if typeDef.type == "array":
    assert value is array
    for item in value:
      validateType(typeDef.items, item)
  else if typeDef.type == "object":
    assert value is object
    for propName, propDef in typeDef.properties:
      if propName in value:
        validateType(propDef, value[propName])
```

## Mustache Rendering

Use a standard Mustache library for your language:

- **Go**: `github.com/cbroglie/mustache`
- **JavaScript**: `mustache.js`
- **Python**: `pystache` or `chevron`
- **Rust**: `mustache`
- **Java**: `mustache.java`

### Template Context

Pass fragment inputs directly as the template context:

```
inputs = {
  "path": "AGENTS.md",
  "focus": "build system"
}

template = "Read {{path}}{{#focus}} and focus on {{focus}}{{/focus}}"
rendered = mustache.render(template, inputs)
```

### Default Values

Apply defaults before rendering:

```
function applyDefaults(inputDefs, providedInputs):
  result = copy(providedInputs)
  for name, def in inputDefs:
    if name not in result and "default" in def:
      result[name] = def.default
  return result
```

## Error Handling

### Validation Errors

Provide clear, actionable error messages:

```
Error: Missing required field 'metadata.id' in Prompt resource at prompts.yml
Error: metadata.id 'my prompt' does not match pattern ^[a-zA-Z0-9_-]+$
Error: Fragment 'read-file' referenced in Prompt 'implement' not found in spec.fragments
Error: Required input 'path' not provided for fragment 'read-file'
Error: Input 'count' expects type 'number', got 'string'
```

### Error Context

Include file paths and line numbers when possible:

```
Error in prompts.yml:
  Line 8: Fragment 'read-file' not found in spec.fragments
```

## Performance Considerations

### Caching

- Cache parsed resources by file path and modification time
- Cache resolved fragment bodies
- Invalidate cache on file changes

### Lazy Loading

- Load resources on-demand rather than all at once
- Resolve fragments only when needed

### Validation

- Validate once at load time, not on every access
- Pre-compile JSON schemas for faster validation

### Resource Limits

Implementations should enforce limits to prevent abuse:

- **Array size**: Limit arrays in templates (recommended: 1000 items)
- **Nesting depth**: Limit object nesting (recommended: 10 levels)
- **Template timeout**: Timeout template rendering (recommended: 5 seconds)
- **File size**: Limit resource file size (recommended: 1MB)

These limits should be configurable.

## Testing

Implementations should test against:

1. **Schema test suite**: `/schema/draft/tests/`
2. **Examples**: `/examples/draft/`
3. **Edge cases**: Empty collections, missing fragments, circular references (should error)

## Conformance

A conforming implementation must:

1. Validate all resources against JSON Schema
2. Resolve fragments from `spec.fragments` within the same resource
3. Render Mustache templates correctly
4. Validate fragment inputs (types and required fields)
5. Fail with clear errors for invalid resources
6. Pass all tests in the schema test suite
