# Security Guidelines

## Mandatory Security Checks

Before ANY commit:
- [ ] No hardcoded secrets (API keys, passwords, tokens)
- [ ] All user inputs validated
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (sanitized HTML)
- [ ] CSRF protection enabled
- [ ] Authentication/authorization verified
- [ ] Rate limiting on all endpoints
- [ ] Error messages don't leak sensitive data

## Secret Management

```text
// NEVER: Hardcoded secrets
apiKey = "sk-proj-xxxxx"

// ALWAYS: Environment variables
apiKey = getEnvVar("OPENAI_API_KEY")

if (!apiKey) {
  throw Error("OPENAI_API_KEY not configured")
}
```

## Security Response Protocol

If security issue found:
1. STOP immediately
2. Use **security-reviewer** agent
3. Fix CRITICAL issues before continuing
4. Rotate any exposed secrets
5. Review entire codebase for similar issues

---

## Language-Specific Examples

- [JavaScript/TypeScript](languages/javascript/security.md) - process.env, dotenv, DOMPurify, Prisma parameterized queries
- [Python](languages/python/security.md) - os.environ, python-dotenv, SQLAlchemy, Jinja2 escaping
- [Go](languages/go/security.md) - os.Getenv, Viper, database/sql placeholders, html/template
- [Rust](languages/rust/security.md) - std::env, dotenvy, sqlx macros, askama escaping
