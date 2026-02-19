# Skill and Skillset Resource Kinds

## Resolution

**Agent Skills are out of scope for the AI Resource Specification.**

See [ADR-003](ADRs/003-skills-out-of-scope.md) for the complete rationale.

## Summary

After thorough research, we determined that Agent Skills already solve the problem that AI Resource Spec aims to solve: providing a unified, portable standard with wide adoption. Adding Skills to this spec would create unnecessary abstraction over an already-successful standard.

**AI Resource Spec focuses on:** Unifying fragmented prompt and rule formats across tools  
**Agent Skills focuses on:** Packaging procedural knowledge and workflows

These are complementary standards, not overlapping ones.

---

## Original Goal (Archived)
Define Skill and Skillset as new resource kinds in the AI Resource Specification, compatible with the Agent Skills format while maintaining consistency with existing Prompt/Rule patterns.

## Background

**Existing resource kinds**: Prompt, Promptset, Rule, Ruleset
- Single YAML files with apiVersion/kind/metadata/spec envelope
- Inline fragments for composition
- Body field supports strings, arrays, and fragment references

**Agent Skills format**: 
- Directory with SKILL.md (YAML frontmatter + Markdown body)
- Optional scripts/, references/, assets/ directories
- Progressive disclosure model (metadata → instructions → resources)
- Spec: https://agentskills.io/specification

## Open Design Questions

### 1. Resource Structure: Inline vs Reference

**Option A: Inline Everything (like Prompt/Rule)**
```yaml
apiVersion: ai-resource/draft
kind: Skill
metadata:
  id: pdf-processing
  name: PDF Processing
spec:
  discovery:
    description: "Extract text from PDFs"
    compatibility: "Requires pdfplumber"
  instructions: |
    ## When to use
    Use when working with PDF files...
  scripts:
    extract.py: |
      #!/usr/bin/env python3
      # Script content inline
  references:
    REFERENCE.md: |
      # Reference content inline
```

**Option B: Reference External Files**
```yaml
apiVersion: ai-resource/draft
kind: Skill
metadata:
  id: pdf-processing
spec:
  discovery:
    description: "Extract text from PDFs"
  instructions: |
    ## When to use
    ...
  scripts:
    - extract.py      # References external file
  references:
    - REFERENCE.md    # References external file
```

**Option C: Hybrid (inline instructions, reference assets)**
```yaml
apiVersion: ai-resource/draft
kind: Skill
metadata:
  id: pdf-processing
spec:
  discovery:
    description: "Extract text from PDFs"
  instructions: |
    ## When to use
    ...
  scripts:
    - path: extract.py
      inline: |
        #!/usr/bin/env python3
        ...
  references:
    - path: REFERENCE.md
      inline: |
        # Reference
        ...
```

**Question**: How do we handle multi-file skills in a single-file resource format?

**Considerations**:
- Inline: Self-contained, but large files become unwieldy
- Reference: Cleaner, but breaks single-file portability
- Hybrid: Flexible, but more complex

### 2. Skillset Structure: Collection vs Nested

**Option A: Flat Collection (like Promptset/Ruleset)**
```yaml
apiVersion: ai-resource/draft
kind: Skillset
metadata:
  id: document-processing
spec:
  skills:
    pdf-processing:
      discovery: {...}
      instructions: |
        ...
    word-processing:
      discovery: {...}
      instructions: |
        ...
```

**Option B: Nested Skills with Shared Resources**
```yaml
apiVersion: ai-resource/draft
kind: Skillset
metadata:
  id: document-processing
spec:
  shared:
    scripts:
      - common.py
    references:
      - SHARED.md
  skills:
    pdf-processing:
      discovery: {...}
      instructions: |
        ...
      scripts:
        - extract.py
    word-processing:
      discovery: {...}
      instructions: |
        ...
```

**Question**: Do skillsets need shared resources, or is each skill independent?

### 3. Discovery Metadata: Separate or Embedded?

Agent Skills use frontmatter for discovery:
```yaml
---
name: pdf-processing
description: Extract text from PDFs
compatibility: Requires pdfplumber
---
```

**Option A: Separate discovery field**
```yaml
spec:
  discovery:
    description: "Extract text from PDFs"
    compatibility: "Requires pdfplumber"
  instructions: |
    ...
```

**Option B: Embed in instructions (like Agent Skills)**
```yaml
spec:
  instructions: |
    ---
    name: pdf-processing
    description: Extract text from PDFs
    ---
    
    ## When to use
    ...
```

**Option C: Use metadata + spec fields**
```yaml
metadata:
  id: pdf-processing
  name: PDF Processing
  description: Extract text from PDFs  # Discovery description
spec:
  compatibility: "Requires pdfplumber"
  instructions: |
    ## When to use
    ...
```

**Question**: Where does discovery metadata live? How do we support progressive disclosure?

### 4. Fragments in Skills

Should skills support fragments like Prompts/Rules?

```yaml
apiVersion: ai-resource/draft
kind: Skill
metadata:
  id: pdf-processing
spec:
  fragments:
    check-deps:
      inputs:
        tool:
          type: string
          required: true
      body: "Ensure {{tool}} is installed"
  instructions:
    - fragment: check-deps
      inputs:
        tool: pdfplumber
    - "Extract text from PDF files..."
```

**Question**: Do skills need fragment composition, or is that overkill?

### 5. Scripts and Assets: Inline or Metadata?

**Option A: Inline with encoding**
```yaml
spec:
  scripts:
    - name: extract.py
      language: python
      content: |
        #!/usr/bin/env python3
        import pdfplumber
        ...
  assets:
    - name: template.pdf
      encoding: base64
      content: "JVBERi0xLjQK..."
```

**Option B: Reference with metadata**
```yaml
spec:
  scripts:
    - path: scripts/extract.py
      language: python
      executable: true
  assets:
    - path: assets/template.pdf
      type: application/pdf
```

**Option C: Simple list (no metadata)**
```yaml
spec:
  scripts:
    - scripts/extract.py
  references:
    - references/REFERENCE.md
  assets:
    - assets/template.pdf
```

**Question**: How much metadata do we need for bundled files?

### 6. Compatibility with Agent Skills Format

**Question**: Should a Skill resource be compilable to SKILL.md format?

If yes, we need:
- Discovery metadata → YAML frontmatter
- Instructions → Markdown body
- Scripts/references/assets → Directory structure

**Compilation example**:
```yaml
# Input: Skill resource
apiVersion: ai-resource/draft
kind: Skill
metadata:
  id: pdf-processing
spec:
  discovery:
    description: "Extract text from PDFs"
  instructions: |
    ## When to use
    ...

# Output: SKILL.md
---
name: pdf-processing
description: Extract text from PDFs
---

## When to use
...
```

**Question**: Is SKILL.md compilation a requirement, or just one possible output format?

### 7. Enforcement and Scope (from Rules)

Should skills support enforcement levels and scope like Rules?

```yaml
apiVersion: ai-resource/draft
kind: Skill
metadata:
  id: pdf-processing
spec:
  enforcement: should      # may|should|must
  scope:
    - files: ["**/*.pdf"]
  discovery: {...}
  instructions: |
    ...
```

**Question**: Do skills need enforcement/scope, or are they always optional capabilities?

### 8. Body Field Consistency

Current resources use `body` field. Should skills?

**Option A: Use body (consistent)**
```yaml
spec:
  discovery: {...}
  body: |
    ## When to use
    ...
```

**Option B: Use instructions (clearer)**
```yaml
spec:
  discovery: {...}
  instructions: |
    ## When to use
    ...
```

**Question**: Consistency vs clarity?

## Proposed Minimal Schema (Draft)

### Skill Resource

```yaml
apiVersion: ai-resource/draft
kind: Skill
metadata:
  id: pdf-processing
  name: PDF Processing
  description: Extract text and tables from PDF files
spec:
  # Optional: Discovery metadata for progressive disclosure
  compatibility: "Requires pdfplumber, python3"
  
  # Optional: Inline fragments
  fragments:
    check-tool:
      inputs:
        tool: {type: string, required: true}
      body: "Ensure {{tool}} is installed"
  
  # Required: Instructions (string or array with fragments)
  body: |
    ## When to use this skill
    Use when working with PDF files...
    
    ## How to extract text
    Run: scripts/extract.py <file>
  
  # Optional: Referenced files (implementation-defined how these are bundled)
  scripts:
    - scripts/extract.py
  references:
    - references/REFERENCE.md
  assets:
    - assets/template.pdf
```

### Skillset Resource

```yaml
apiVersion: ai-resource/draft
kind: Skillset
metadata:
  id: document-processing
  name: Document Processing Skills
spec:
  # Optional: Shared fragments across skills
  fragments:
    check-tool:
      inputs:
        tool: {type: string, required: true}
      body: "Ensure {{tool}} is installed"
  
  # Required: Map of skills
  skills:
    pdf-processing:
      name: PDF Processing
      description: Extract text from PDFs
      compatibility: "Requires pdfplumber"
      body: |
        ## When to use
        ...
      scripts:
        - scripts/pdf/extract.py
    
    word-processing:
      name: Word Processing
      description: Process Word documents
      body: |
        ## When to use
        ...
```

## Key Decisions Needed

1. **Multi-file handling**: Inline vs reference vs hybrid?
2. **Skillset structure**: Flat collection vs nested with shared resources?
3. **Discovery metadata**: Separate field vs embedded vs metadata-only?
4. **Fragment support**: Yes or no?
5. **Script metadata**: Full metadata vs simple paths?
6. **SKILL.md compilation**: Required output format or optional?
7. **Enforcement/scope**: Inherit from Rules or omit?
8. **Body field**: Use `body` or `instructions`?

## Constraints

- Must follow envelope structure (apiVersion, kind, metadata, spec)
- Should be consistent with Prompt/Promptset/Rule/Ruleset patterns
- Must support single-file YAML format (primary distribution)
- Should enable compilation to Agent Skills SKILL.md format
- Must support versioning and validation

## Next Steps

1. Resolve design questions above
2. Draft complete specification in spec/draft/
3. Create JSON schemas in schema/draft/
4. Add examples in examples/draft/
5. Write ADR documenting key decisions
