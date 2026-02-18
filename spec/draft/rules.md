# Rules (draft)

Rules define constraints, recommendations, or suggestions for AI behavior. They include enforcement levels, optional scope targeting, and optional priority.

## Enforcement Levels

Rules support three enforcement levels that indicate the strength of the constraint.

### may

**Meaning:** Optional suggestion  
**Expectation:** AI may consider this guidance but is free to ignore it

```yaml
enforcement: may
body: "Consider using descriptive variable names"
```

### should

**Meaning:** Recommended practice  
**Expectation:** AI should follow this guidance unless there's a good reason not to

```yaml
enforcement: should
body: "Use descriptive variable names"
```

### must

**Meaning:** Required constraint  
**Expectation:** AI must follow this guidance without exception

```yaml
enforcement: must
body: "Never expose API keys in code"
```

## Scope

Rules can target specific files using glob patterns. Scope is optional; rules without scope apply globally.

### Structure

```yaml
scope:
  - files: [<pattern>, <pattern>, ...]
  - files: [<pattern>]
```

### Patterns

Scope uses glob patterns for file matching:

- `**/*.py` - All Python files recursively
- `src/**/*.ts` - All TypeScript files under src/
- `*.json` - JSON files in root directory
- `test/**/*` - All files under test/

### Examples

**Single pattern:**
```yaml
scope:
  - files: ["**/*.py"]
```

**Multiple patterns in one entry:**
```yaml
scope:
  - files: ["**/*.ts", "**/*.tsx"]
```

**Multiple entries:**
```yaml
scope:
  - files: ["src/**/*.py"]
  - files: ["lib/**/*.py"]
```

## Priority

Priority determines rule ordering when multiple rules apply. Higher numbers indicate higher priority.

**Type:** number  
**Default:** 100  
**Range:** Any integer

### Example

```yaml
rules:
  critical-security:
    priority: 200
    enforcement: must
    body: "Never expose secrets"
  
  code-style:
    priority: 50
    enforcement: should
    body: "Use consistent formatting"
```

## Complete Example

```yaml
apiVersion: ai-resource/draft
kind: Rule
metadata:
  id: no-hardcoded-secrets
  name: No Hardcoded Secrets
  description: Prevents hardcoding secrets in source code
spec:
  enforcement: must
  scope:
    - files: ["**/*.py", "**/*.js", "**/*.ts"]
  body: |
    Never hardcode secrets, API keys, or credentials in source code.
    Use environment variables or secure secret management instead.
```

## Ruleset Example

```yaml
apiVersion: ai-resource/draft
kind: Ruleset
metadata:
  id: security-rules
  name: Security Rules
spec:
  rules:
    no-secrets:
      priority: 200
      enforcement: must
      scope:
        - files: ["**/*.py", "**/*.js"]
      body: "Never hardcode secrets"
    
    validate-input:
      priority: 150
      enforcement: must
      body: "Always validate user input"
    
    use-https:
      priority: 100
      enforcement: should
      body: "Prefer HTTPS for external requests"
```
