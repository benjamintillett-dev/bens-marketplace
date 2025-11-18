---
name: code-documentation
description: Automatically generates and maintains high-quality code documentation. Invoked when writing new functions/classes, modifying public APIs, or when code complexity warrants explanation. Follows language-specific documentation conventions (JSDoc, docstrings, etc.) and focuses on intent over implementation details.
allowed-tools: ["Read", "Edit", "Grep", "Glob"]
---

# Code Documentation Skill

You are equipped with expert knowledge of documentation best practices across all programming languages.

## When This Skill Activates

Claude will invoke this skill when:
- Writing new functions, classes, or modules
- Modifying public APIs
- Working with complex logic that needs explanation
- User explicitly requests documentation

## Documentation Standards by Language

### Python
```python
def function_name(param1: str, param2: int) -> bool:
    """Brief one-line summary.

    More detailed explanation if needed. Describe the purpose,
    not the implementation mechanics.

    Args:
        param1: Description of first parameter
        param2: Description of second parameter

    Returns:
        Description of return value

    Raises:
        ValueError: When this error occurs

    Example:
        >>> function_name("test", 42)
        True
    """
```

### JavaScript/TypeScript
```javascript
/**
 * Brief one-line summary.
 *
 * More detailed explanation if needed.
 *
 * @param {string} param1 - Description of first parameter
 * @param {number} param2 - Description of second parameter
 * @returns {boolean} Description of return value
 * @throws {Error} When this error occurs
 *
 * @example
 * functionName("test", 42);
 * // => true
 */
```

### Go
```go
// FunctionName brief summary.
//
// More detailed explanation if needed. Describe purpose and behavior.
// Document any important constraints or edge cases.
//
// Parameters:
//   - param1: description
//   - param2: description
//
// Returns an error if validation fails.
```

### Rust
```rust
/// Brief one-line summary.
///
/// More detailed explanation if needed.
///
/// # Arguments
///
/// * `param1` - Description of first parameter
/// * `param2` - Description of second parameter
///
/// # Returns
///
/// Description of return value
///
/// # Errors
///
/// Returns error when...
///
/// # Examples
///
/// ```
/// let result = function_name("test", 42)?;
/// assert_eq!(result, true);
/// ```
```

## Documentation Principles

1. **Clarity First**: Write for someone unfamiliar with the code
2. **Intent Over Implementation**: Explain WHY, not HOW
3. **Conciseness**: Be clear but brief
4. **Examples**: Include for complex APIs
5. **Update on Change**: Keep docs in sync with code
6. **Avoid Obviousness**: Don't document trivial code

## What to Document

**Always Document**:
- Public APIs (functions, classes, modules)
- Complex algorithms or business logic
- Non-obvious behavior or side effects
- Parameters, return values, exceptions
- Usage examples for complex functions

**Don't Document**:
- Self-explanatory code
- Private implementation details
- Every single line (avoid comment clutter)

## Inline Comments

Use sparingly for:
- Complex logic that isn't obvious
- Workarounds or non-obvious solutions
- Important constraints or assumptions
- TODOs and FIXMEs (with context)

```python
# Use rate limiting here because the API has a 100req/min limit
# See: https://docs.example.com/api/limits
await rate_limiter.wait()
```

## When to Skip

Skip documentation for:
- Getters/setters with obvious behavior
- Simple utility functions (e.g., `isEmpty()`)
- Test code (unless testing strategy is complex)
- Temporary/prototype code

## Quality Checks

Before considering documentation complete:
- [ ] Does it explain WHY, not just WHAT?
- [ ] Can a new developer understand without reading implementation?
- [ ] Are all parameters and return values described?
- [ ] Are error conditions documented?
- [ ] Are there examples for complex usage?
- [ ] Is it accurate (matches actual behavior)?

Remember: Good documentation is maintainable documentation. When in doubt, err on the side of clarity.
