# Security Guidelines - JavaScript/TypeScript

## Secret Management with Environment Variables

```typescript
// NEVER: Hardcoded secrets
const apiKey = "sk-proj-xxxxx"

// ALWAYS: Environment variables
const apiKey = process.env.OPENAI_API_KEY

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

### Loading Environment Variables

Using dotenv:

```typescript
import 'dotenv/config'

const config = {
  apiKey: process.env.API_KEY,
  dbUrl: process.env.DATABASE_URL,
  port: parseInt(process.env.PORT || '3000', 10)
}
```

Using Zod for validated config:

```typescript
import { z } from 'zod'

const envSchema = z.object({
  API_KEY: z.string().min(1),
  DATABASE_URL: z.string().url(),
  PORT: z.string().transform(Number).default('3000')
})

const env = envSchema.parse(process.env)
```

## XSS Prevention

### React (automatic escaping)

```tsx
// Safe: React escapes by default
<div>{userInput}</div>

// DANGEROUS: Never use dangerouslySetInnerHTML with user input
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

### Sanitizing HTML

```typescript
import DOMPurify from 'dompurify'

const sanitized = DOMPurify.sanitize(userInput)
```

## SQL Injection Prevention

Use parameterized queries:

```typescript
// WRONG: String concatenation
const query = `SELECT * FROM users WHERE id = '${userId}'`

// CORRECT: Parameterized query
const user = await db.query('SELECT * FROM users WHERE id = $1', [userId])

// CORRECT: ORM (Prisma)
const user = await prisma.user.findUnique({ where: { id: userId } })
```
