# Schema Test Suite

Test fixtures for validating JSON Schema conformance.

## Structure

- `valid/` - Resources that must pass validation
- `invalid/` - Resources that must fail validation

## Valid Test Cases

### Fragment
- `fragment-basic.yml` - Basic fragment with simple inputs
- `fragment-complex-inputs.yml` - Fragment with array and object inputs

### Prompt
- `prompt-simple.yml` - Simple string body
- `prompt-with-fragments.yml` - Body with fragment references

### Promptset
- `promptset-basic.yml` - Collection with multiple prompts

### Rule
- `rule-basic.yml` - Rule with enforcement and scope

### Ruleset
- `ruleset-basic.yml` - Collection with multiple rules

## Invalid Test Cases

### Fragment
- `fragment-invalid-id.yml` - ID with spaces (violates pattern)
- `fragment-missing-body.yml` - Missing required body field

### Prompt
- `prompt-missing-body.yml` - Missing required body field

### Promptset
- `promptset-empty.yml` - Empty collection (violates minProperties)

### Rule
- `rule-invalid-enforcement.yml` - Invalid enforcement value

### Ruleset
- `ruleset-empty.yml` - Empty collection (violates minProperties)

## Running Tests

Implementations should validate all files in `valid/` pass and all files in `invalid/` fail with appropriate error messages.
