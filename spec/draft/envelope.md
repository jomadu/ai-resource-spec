# Resource Envelope (draft)

All AI resources follow a Kubernetes-style envelope structure with four required top-level fields.

## Structure

```yaml
apiVersion: ai-resource/draft    # Required: API version
kind: <ResourceKind>             # Required: Resource type
metadata:                        # Required: Resource metadata
  id: <identifier>               # Required: Machine identifier
  name: <human-name>             # Optional: Human-readable name
  description: <text>            # Optional: Description
spec:                            # Required: Resource-specific content
  # ... kind-specific fields
```

## Fields

### apiVersion

**Type:** string  
**Required:** Yes  
**Format:** `ai-resource/draft`

Specifies the API version. Current version is `ai-resource/draft`.

### kind

**Type:** string  
**Required:** Yes  
**Values:** `Prompt`, `Promptset`, `Rule`, `Ruleset`

Specifies the resource type.

### metadata

**Type:** object  
**Required:** Yes

Contains resource metadata.

#### metadata.id

**Type:** string  
**Required:** Yes  
**Pattern:** `^[a-zA-Z0-9_-]+$`

Machine identifier for the resource. Must be unique within a directory namespace.

#### metadata.name

**Type:** string  
**Required:** No

Human-readable name for the resource.

#### metadata.description

**Type:** string  
**Required:** No

Description of the resource's purpose or behavior.

### spec

**Type:** object  
**Required:** Yes

Contains resource-specific configuration. Structure varies by `kind`.

## Examples

### Minimal Resource

```yaml
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: summarize
spec:
  body: "Summarize the code"
```

### Complete Resource

```yaml
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: summarize
  name: Summarize Code
  description: Generates a concise summary of code for review
spec:
  body: "Summarize the following code for review"
```

## Validation

- All four top-level fields (`apiVersion`, `kind`, `metadata`, `spec`) are required
- `metadata.id` must match pattern `^[a-zA-Z0-9_-]+$`
- `metadata.id` must be unique within the directory namespace
- `spec` structure must conform to the schema for the specified `kind`

## Multi-Document YAML

YAML files may contain multiple resources separated by `---` document separators.

### Semantics

- Each document is an independent resource
- `metadata.id` must be unique within the file
- Fragments defined in `spec.fragments` are scoped to their document (cannot reference across documents)
- Different resource kinds may appear in the same file

### Example

```yaml
---
apiVersion: ai-resource/draft
kind: Prompt
metadata:
  id: summarize
spec:
  body: "Summarize the following code"
---
apiVersion: ai-resource/draft
kind: Rule
metadata:
  id: no-secrets
spec:
  enforcement: must
  body: "Never hardcode secrets"
---
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
      body: "Read {{path}}"
  body:
    - fragment: read-file
      inputs:
        path: AGENTS.md
    - "Implement the feature"
```
