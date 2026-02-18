# Refactoring Complete: v1 → draft, Standalone → Inline Fragments

## Summary

Successfully completed the two-part refactoring of the AI Resource Specification:

1. **Renamed v1 to draft** throughout the repository
2. **Removed standalone Fragments**, added inline fragment definitions

## Changes Made

### 1. Directory Structure

**Renamed:**
- `spec/v1/` → `spec/draft/`
- `schema/v1/` → `schema/draft/`
- `examples/v1/` → `examples/draft/`

### 2. Version References

**Updated all occurrences:**
- `apiVersion: ai-resource/v1` → `apiVersion: ai-resource/draft`
- Schema `$id` URLs: `/v1/` → `/draft/`
- Documentation links and references

### 3. Resource Kinds

**Removed:**
- `kind: Fragment` as a standalone resource type
- Fragment.schema.json

**Updated:**
- envelope.md: Removed Fragment from kind list (5 → 4 kinds)
- resource-kinds.md: Removed Fragment section, updated all other kinds

### 4. Inline Fragments

**Added `spec.fragments` field to:**
- Prompt
- Promptset
- Rule
- Ruleset

**Structure:**
```yaml
spec:
  fragments:
    <fragment-id>:
      inputs:          # Optional
        <name>:
          type: <type>
          required: <bool>
          default: <value>
      body: <template>
```

### 5. Specification Documents

**Updated:**
- `spec/draft/envelope.md` - Removed Fragment, updated version
- `spec/draft/resource-kinds.md` - Added inline fragments to all kinds, removed Fragment section
- `spec/draft/fragments.md` - Completely rewritten for inline model
- `spec/draft/rules.md` - Updated version references
- `spec/draft/validation.md` - Updated for resource-scoped validation
- `spec/draft/implementation.md` - Rewritten for inline fragment resolution

### 6. JSON Schemas

**Deleted:**
- `schema/draft/Fragment.schema.json`

**Updated all schemas with:**
- `apiVersion: "ai-resource/draft"`
- Schema `$id`: `https://ai-resource.dev/schema/draft/...`
- Added `spec.fragments` field (optional)
- Added `fragmentDefinition` and `inputDefinition` definitions

### 7. Examples

**Deleted standalone fragment files:**
- `examples/draft/basic/fragment.yml`
- `examples/draft/composition/read-file.yml`
- `examples/draft/composition/coding-standards.yml`
- `examples/draft/real-world/context-setup.yml`
- `examples/draft/real-world/analyze-files.yml`

**Updated to use inline fragments:**
- `examples/draft/composition/prompt.yml`
- `examples/draft/composition/promptset.yml`
- `examples/draft/real-world/code-review.yml`

**Updated version references:**
- All basic examples
- `examples/draft/real-world/python-rules.yml`

### 8. Tests

**Deleted:**
- `schema/draft/tests/valid/fragment-basic.yml`
- `schema/draft/tests/valid/fragment-complex-inputs.yml`
- `schema/draft/tests/invalid/fragment-invalid-id.yml`
- `schema/draft/tests/invalid/fragment-missing-body.yml`

**Added:**
- `schema/draft/tests/valid/promptset-with-fragments.yml`
- `schema/draft/tests/valid/ruleset-with-fragments.yml`
- `schema/draft/tests/valid/prompt-complex-fragment-inputs.yml`
- `schema/draft/tests/invalid/prompt-invalid-fragment-id.yml`
- `schema/draft/tests/invalid/prompt-fragment-missing-body.yml`

**Updated:**
- All test files: `ai-resource/v1` → `ai-resource/draft`
- `prompt-with-fragments.yml` - Updated to use inline fragments

### 9. ADRs

**Created:**
- `ADRs/002-inline-fragments.md` - Documents decision to move to inline fragments

**Updated:**
- `ADRs/001-fragments-as-resources.md` - Marked as superseded by ADR-002

### 10. Documentation

**Updated:**
- `README.md` - Updated version references, examples, documentation links
- `examples/draft/README.md` - Updated for inline fragment model

## Key Design Changes

### Fragment Scope

**Before:** Directory-scoped
- Fragments resolved by `metadata.id` from same directory
- Cross-file dependencies
- Shared namespace

**After:** Resource-scoped
- Fragments resolved from `spec.fragments` within same resource
- Self-contained resources
- No cross-file sharing

### Fragment Resolution

**Before:**
1. Load all resources in directory
2. Index fragments by `metadata.id`
3. Resolve fragment references against directory index

**After:**
1. Load resource
2. Resolve fragment references against `spec.fragments` in same resource

### Benefits

✅ Self-contained resources (everything in one file)
✅ Simpler specification (4 kinds instead of 5)
✅ Simpler implementation (no directory loading/indexing)
✅ Clearer scope (resource-scoped, not directory-scoped)
✅ Still supports DRY within Rulesets/Promptsets

### Tradeoffs

❌ Can't share fragments across resources
❌ Duplication if same fragment needed in multiple resources

**Acceptable because:**
- Most resources don't need shared fragments
- If needed, copy the content (it's just text)
- Keeps the spec simple and focused

## Verification

All changes verified:
- ✅ No remaining `v1` references (except in TASK.md and ADRs)
- ✅ No remaining `kind: Fragment` (except in TASK.md and ADRs)
- ✅ Fragment.schema.json deleted
- ✅ All schemas have `spec.fragments` field
- ✅ All schemas use `ai-resource/draft`
- ✅ All examples updated
- ✅ All tests updated
- ✅ Documentation updated

## Status

**Complete.** The specification is now in draft status with inline fragments. Ready for v1 release after validation and feedback.
