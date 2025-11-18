---
description: Generate or improve documentation for code
argument-hint: "[file-pattern or scope]"
model: sonnet
---

# Documentation Generator

Generate comprehensive, maintainable documentation for your code.

## Scope
$ARGUMENTS

## Documentation Strategy

1. **Identify target code**:
   - If file pattern provided: document those files
   - If "recent" or no args: document recent changes (git diff)
   - If "missing": find undocumented public APIs

2. **Generate appropriate documentation**:
   - **Functions/Methods**: Purpose, parameters, return values, exceptions, examples
   - **Classes**: Responsibility, usage patterns, key methods
   - **Modules/Packages**: Overview, main exports, usage guide
   - **Complex Logic**: Inline comments explaining "why", not "what"

3. **Follow language conventions**:
   - Python: docstrings (Google/NumPy style)
   - JavaScript/TypeScript: JSDoc
   - Go: standard comments
   - Rust: Rustdoc
   - Java: JavaDoc
   - Ruby: YARD

## Documentation Principles

- **Clarity**: Write for someone unfamiliar with the code
- **Accuracy**: Ensure docs match implementation exactly
- **Conciseness**: Be clear but brief - avoid unnecessary verbosity
- **Examples**: Include usage examples for complex APIs
- **Why over What**: Explain reasoning and context, not just mechanics

## Output

- Show what documentation was added/improved
- Highlight any ambiguities that need human clarification
- Suggest README updates if public API changed

Avoid over-commenting obvious code. Focus on intent, constraints, and non-obvious behavior.
