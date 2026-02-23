# TODO

## TASK-001
- Priority: 1
- Status: DONE
- Dependencies: []
- Description: Define multi-document YAML semantics in envelope.md

Add section to `spec/draft/envelope.md` specifying:
- Each document is an independent resource
- `metadata.id` must be unique within the file
- Fragments are scoped to their document (cannot reference across documents)
- Different resource kinds may be in the same file
- Include example showing multiple resources in one file

Reference: ISSUES.md #7

---

## TASK-002
- Priority: 1
- Status: DONE
- Dependencies: []
- Description: Split validation.md into schema/semantic phases with detailed rules

Update `spec/draft/validation.md`:
- Split into "Schema Validation" and "Semantic Validation" sections
- Document validation order (schema first, then semantic)
- Specify error message requirements (field paths, clear descriptions)
- Add comprehensive list of validation rules for each phase
- Add examples for each validation rule
- Reference ai-resource-core-go specs as implementation example

**Phase 1: Schema Validation** (structural)
- Required fields present
- Field types correct
- Patterns match (metadata.id, collection keys)
- Enums valid (enforcement levels)
- Collections have minimum items

**Phase 2: Semantic Validation** (logical)
- Fragment references exist in spec.fragments
- Required fragment inputs provided
- No undefined fragment inputs
- Collection keys match pattern ^[a-zA-Z0-9_-]+$
- Body format valid (string or array of strings/fragment refs)

Reference: ISSUES.md #4

---

## TASK-003
- Priority: 2
- Status: DONE
- Dependencies: []
- Description: Add concrete enforcement semantics to implementation.md

Add section to `spec/draft/implementation.md` defining how AI systems should handle enforcement levels:

```markdown
## Enforcement Level Handling

Implementations should interpret enforcement levels as follows:

- **may**: Include rule in context. No validation or warnings.
- **should**: Include rule in context. Implementations MAY warn users when violations are detected.
- **must**: Include rule in context. Implementations SHOULD prevent or block actions that violate the rule.

Note: Enforcement is best-effort. AI systems cannot guarantee perfect compliance.
```

Reference: ISSUES.md #5

---

## TASK-004
- Priority: 3
- Status: TODO
- Dependencies: []
- Description: Document version migration strategy

Add section to README.md documenting:
1. Draft will be supported for 6 months after v1 release
2. Implementations should warn when loading draft resources
3. Plan to provide `tools/migrate` CLI tool to convert draft → v1
4. Deprecation timeline and compatibility guarantees

Reference: ISSUES.md #8
