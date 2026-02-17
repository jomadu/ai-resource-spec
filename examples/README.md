# Examples

This directory contains example AI resources demonstrating various use cases.

## Structure

- `basic/` - One example per resource kind
- `composition/` - Fragment composition examples
- `real-world/` - Practical use cases

## Basic Examples

Simple examples showing the structure of each resource kind:

- `prompt.yml` - Single prompt
- `promptset.yml` - Collection of prompts
- `rule.yml` - Single rule with enforcement and scope
- `ruleset.yml` - Collection of rules
- `fragment.yml` - Reusable template with inputs

## Composition Examples

Demonstrates fragment composition across a directory:

- `read-file.yml` - Fragment for reading files
- `coding-standards.yml` - Fragment for coding standards
- `prompt.yml` - Prompt using fragments
- `promptset.yml` - Promptset using fragments

Shows how fragments are referenced by ID and resolved from the same directory.

## Real-World Examples

Practical use cases:

### Code Review Workflow
- `context-setup.yml` - Context fragment
- `analyze-files.yml` - File analysis fragment with array inputs
- `code-review.yml` - Complete code review promptset

### Python Best Practices
- `python-rules.yml` - Comprehensive Python ruleset with priorities

## Usage

Each directory is self-contained. Resources reference fragments by `metadata.id` within the same directory.

To use these examples:
1. Copy the directory to your project
2. Modify resource IDs and content as needed
3. Ensure fragment IDs match references in `body` fields
