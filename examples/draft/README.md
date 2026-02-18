# Examples

This directory contains example AI resources demonstrating various use cases.

## Structure

- `basic/` - One example per resource kind
- `composition/` - Inline fragment composition examples
- `real-world/` - Practical use cases

## Basic Examples

Simple examples showing the structure of each resource kind:

- `prompt.yml` - Single prompt
- `promptset.yml` - Collection of prompts
- `rule.yml` - Single rule with enforcement and scope
- `ruleset.yml` - Collection of rules

## Composition Examples

Demonstrates inline fragment composition:

- `prompt.yml` - Prompt with inline fragments
- `promptset.yml` - Promptset with inline fragments shared across prompts

Shows how fragments are defined in `spec.fragments` and referenced in `body` fields.

## Real-World Examples

Practical use cases:

### Code Review Workflow
- `code-review.yml` - Complete code review promptset with inline fragments for context setup and file analysis

### Python Best Practices
- `python-rules.yml` - Comprehensive Python ruleset with priorities and scopes

## Usage

Each resource is self-contained with inline fragment definitions.

To use these examples:
1. Copy the resource file to your project
2. Modify resource IDs and content as needed
3. Fragments are scoped to the resource - no cross-file dependencies
