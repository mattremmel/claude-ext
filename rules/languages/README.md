# Language-Specific Rules

This directory contains language-specific implementations of the coding rules defined in the parent `rules/` directory.

## Available Languages

| Language | Status | Description |
|----------|--------|-------------|
| [JavaScript/TypeScript](javascript/) | Complete | Full implementations with examples |
| [Python](python/) | Placeholder | TODO checklists for future implementation |
| [Go](go/) | Placeholder | TODO checklists for future implementation |
| [Rust](rust/) | Complete | Full implementations with examples |

## File Structure

Each language directory contains:

- `coding-style.md` - Immutability, error handling, validation patterns
- `patterns.md` - API responses, repositories, common design patterns
- `hooks.md` - Package managers, formatters, linters, debug detection
- `security.md` - Environment variables, injection prevention, escaping
- `testing.md` - Unit testing, E2E testing, mocking, coverage

## Adding a New Language

1. Create a new directory under `languages/`
2. Copy the placeholder structure from an existing language (e.g., `python/`)
3. Replace TODO checklists with actual implementations
4. Update this README to reflect the new language status

## Usage with Claude Code

When working in a specific language, Claude should:

1. Reference the base rules in `rules/*.md` for universal principles
2. Consult `rules/languages/<language>/*.md` for implementation details
3. If a language has placeholder files, apply the base principles using language idioms
