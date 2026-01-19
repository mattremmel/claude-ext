# Coding Style

## Immutability (CRITICAL)

ALWAYS create new objects, NEVER mutate. Return new instances with updated values instead of modifying in place.

## File Organization

MANY SMALL FILES > FEW LARGE FILES:
- High cohesion, low coupling
- 200-400 lines typical, 800 max
- Extract utilities from large components
- Organize by feature/domain, not by type

## Error Handling

ALWAYS handle errors comprehensively:
- Wrap risky operations in try-catch (or language equivalent)
- Log errors with context
- Throw user-friendly error messages

## Input Validation

ALWAYS validate user input using schema validation libraries appropriate to your language.

## Code Quality Checklist

Before marking work complete:
- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Proper error handling
- [ ] No debug statements (console.log, print, etc.)
- [ ] No hardcoded values
- [ ] No mutation (immutable patterns used)

---

## Language-Specific Examples

- [JavaScript/TypeScript](languages/javascript/coding-style.md) - Spread operators, Zod validation, async/await patterns
- [Python](languages/python/coding-style.md) - Dataclasses, Pydantic, exception handling
- [Go](languages/go/coding-style.md) - Value receivers, error returns, validator
- [Rust](languages/rust/coding-style.md) - Ownership, Result types, thiserror
