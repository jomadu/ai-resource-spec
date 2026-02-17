# Validation

All resources must be validated against their JSON Schema definitions.

## Schema Files

JSON schemas are provided for each resource kind:

- `/schema/ai-resource-v1/Prompt.schema.json`
- `/schema/ai-resource-v1/Promptset.schema.json`
- `/schema/ai-resource-v1/Rule.schema.json`
- `/schema/ai-resource-v1/Ruleset.schema.json`
- `/schema/ai-resource-v1/Fragment.schema.json`

## Validation Requirements

### Structural Validation

All resources must validate against their schema:

- Required fields present
- Field types correct
- Patterns match (e.g., `metadata.id` matches `^[a-zA-Z0-9_-]+$`)
- Enums valid (e.g., `enforcement` is `may`, `should`, or `must`)
- Collections have minimum one item

### Semantic Validation

Additional validation beyond schema:

- `metadata.id` unique within directory namespace
- Fragment references resolve to existing fragments
- Fragment input types match fragment definitions
- Required fragment inputs provided

## Error Handling

Implementations must fail validation with clear error messages:

**Missing required field:**
```
Error: Missing required field 'metadata.id' in Prompt resource
```

**Invalid pattern:**
```
Error: metadata.id 'my prompt' does not match pattern ^[a-zA-Z0-9_-]+$
```

**Fragment not found:**
```
Error: Fragment 'read-file' referenced in Prompt 'implement-feature' not found in directory
```

**Missing fragment input:**
```
Error: Required input 'path' not provided for fragment 'read-file' in Prompt 'implement-feature'
```

**Type mismatch:**
```
Error: Input 'count' expects type 'number', got 'string' in fragment 'list-items'
```

## Conformance Testing

Implementations should validate against the test suite:

- Valid examples must pass validation
- Invalid examples must fail validation with appropriate errors
- Fragment resolution must work correctly
- Input validation must enforce types and requirements

## Validation Timing

Implementations may validate at different times:

- **Load time:** Validate when loading resources from disk
- **Composition time:** Validate when resolving fragments
- **Runtime:** Validate when using resources

All validation must occur before resource use.
