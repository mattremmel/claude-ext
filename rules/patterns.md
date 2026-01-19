# Common Patterns

## API Response Format

Standardize API responses with:
- `success`: boolean indicating outcome
- `data`: optional payload on success
- `error`: optional error message on failure
- `meta`: optional pagination/metadata

## Debounce Pattern

Delay execution until input stabilizes. Useful for search inputs, resize handlers, and other high-frequency events.

## Repository Pattern

Abstract data access behind a consistent interface:
- `findAll(filters?)` - Query multiple records
- `findById(id)` - Get single record
- `create(data)` - Insert new record
- `update(id, data)` - Modify existing record
- `delete(id)` - Remove record

## Skeleton Projects

When implementing new functionality:
1. Search for battle-tested skeleton projects
2. Use parallel agents to evaluate options:
   - Security assessment
   - Extensibility analysis
   - Relevance scoring
   - Implementation planning
3. Clone best match as foundation
4. Iterate within proven structure

---

## Language-Specific Examples

- [JavaScript/TypeScript](languages/javascript/patterns.md) - TypeScript interfaces, React hooks, Prisma repositories
- [Python](languages/python/patterns.md) - Dataclasses, SQLAlchemy repositories, decorators
- [Go](languages/go/patterns.md) - Struct definitions, interface-based repositories, functional options
- [Rust](languages/rust/patterns.md) - Serde structs, trait-based repositories, builder pattern
