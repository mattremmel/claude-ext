# Hooks Configuration - JavaScript/TypeScript

## PreToolUse Hooks

### tmux Reminder
Suggests tmux for long-running package manager commands:
- `npm install`, `npm run`, `npm test`
- `pnpm install`, `pnpm run`
- `yarn install`, `yarn run`
- `npx`, `bunx`

## PostToolUse Hooks

### Prettier Auto-Format
Runs Prettier on JS/TS files after editing:
- `.js`, `.jsx`
- `.ts`, `.tsx`
- `.json`
- `.css`, `.scss`

Configuration typically in `.prettierrc`:

```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 100
}
```

### TypeScript Check
Runs `tsc --noEmit` after editing `.ts`/`.tsx` files to catch type errors immediately.

### console.log Warning
Warns when `console.log` statements are found in edited files. Use proper logging instead:

```typescript
// AVOID: console.log
console.log('Debug:', data)

// PREFER: Structured logging
logger.debug('Processing data', { data, context })
```

## Stop Hooks

### console.log Audit
Before session ends, checks all modified files for `console.log` statements that should be removed before committing.

## ESLint Integration

Common rules for JS/TS projects:

```json
{
  "rules": {
    "no-console": "warn",
    "no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn"
  }
}
```
