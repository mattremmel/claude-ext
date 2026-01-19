# Coding Style - JavaScript/TypeScript

## Immutability

Use spread operators to create new objects instead of mutating:

```javascript
// WRONG: Mutation
function updateUser(user, name) {
  user.name = name  // MUTATION!
  return user
}

// CORRECT: Immutability
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

For arrays:

```javascript
// WRONG: Mutation
function addItem(items, item) {
  items.push(item)  // MUTATION!
  return items
}

// CORRECT: Immutability
function addItem(items, item) {
  return [...items, item]
}
```

## Error Handling

Use async/await with try-catch:

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  // Use structured logging in production (not console.error)
  logger.error('Operation failed', { error, context: 'riskyOperation' })
  throw new Error('Detailed user-friendly message')
}
```

For applications without a logger, use a minimal wrapper:

```typescript
const logger = {
  error: (msg: string, meta?: object) => {
    if (process.env.NODE_ENV !== 'test') {
      console.error(JSON.stringify({ level: 'error', msg, ...meta }))
    }
  }
}
```

## Input Validation with Zod

Use Zod for schema validation:

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

Common Zod patterns:

```typescript
// Optional with default
z.string().optional().default('')

// Union types
z.union([z.string(), z.number()])

// Enums
z.enum(['admin', 'user', 'guest'])

// Nested objects
z.object({
  user: z.object({
    name: z.string(),
    email: z.string().email()
  })
})
```
