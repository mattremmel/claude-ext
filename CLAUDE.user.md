# User-Level CLAUDE.md

## Core Philosophy

You are Claude Code. I use specialized agents and skills for complex tasks.

**Key Principles:**

1. **Agent-First**: Delegate to specialized agents for complex work
2. **Parallel Execution**: Use Task tool with multiple agents when possible
3. **Plan Before Execute**: Use Plan Mode for complex operations
4. **Test-Driven**: Write tests before implementation
5. **Security-First**: Never compromise on security

---

## MANDATORY: Before Writing Code

**STOP. Before writing any code, read the language-specific rules:**

| Language | Read These Files |
|----------|------------------|
| JavaScript/TypeScript | `~/.claude/rules/languages/javascript/*.md` |
| Python | `~/.claude/rules/languages/python/*.md` |
| Go | `~/.claude/rules/languages/go/*.md` |
| Rust | `~/.claude/rules/languages/rust/*.md` |

Each language directory contains: `coding-style.md`, `testing.md`, `patterns.md`, `hooks.md`, `security.md`

DO NOT write code without first reading the relevant language rules.

---

## MANDATORY: After Writing/Modifying Code

**STOP. Before doing anything else after modifying code:**

1. **Run `code-reviewer` agent** - for ALL code changes, no exceptions
2. **Run `security-reviewer` agent** - for auth, user input, API endpoints, or sensitive data
3. **Fix all CRITICAL and HIGH issues** before proceeding to tests or commits

DO NOT skip these steps. DO NOT proceed to testing or committing without running these agents first.

---

## Modular Rules

Detailed guidelines are in `~/.claude/rules/` and language specific guidelines are in `~/.claude/rules/languages`:

| Rule File       | Contents                                        |
| --------------- | ----------------------------------------------- |
| security.md     | Security checks, secret management              |
| coding-style.md | Immutability, file organization, error handling |
| testing.md      | TDD workflow, 80% coverage requirement          |
| git-workflow.md | Commit format, PR workflow                      |
| agents.md       | Agent orchestration, when to use which agent    |
| patterns.md     | API response, repository patterns               |
| performance.md  | Model selection, context management             |

---

## Available Agents

Located in `~/.claude/agents/`:

| Agent                | Purpose                          |
| -------------------- | -------------------------------- |
| planner              | Feature implementation planning  |
| architect            | System design and architecture   |
| tdd-guide            | Test-driven development          |
| code-reviewer        | Code review for quality/security |
| security-reviewer    | Security vulnerability analysis  |
| build-error-resolver | Build error resolution           |
| e2e-runner           | Playwright E2E testing           |
| refactor-cleaner     | Dead code cleanup                |
| doc-updater          | Documentation updates            |

---

## Personal Preferences

### Code Style

- No emojis in code, comments, or documentation
- Prefer immutability - never mutate objects or arrays
- Many small files over few large files
- 200-400 lines typical, 800 max per file under normal circumstances

### Git

- Conventional commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`
- Always test locally before committing
- Small, focused commits

### Testing

- TDD: Write tests first
- 80% minimum coverage
- Unit + integration + E2E for critical flows

---

## Success Metrics

You are successful when:

- **Language-specific rules were read** before writing code
- All tests pass (80%+ coverage)
- No security vulnerabilities
- Code is readable and maintainable
- User requirements are met
- **code-reviewer agent was run and issues addressed** (for all code changes)
- **security-reviewer agent was run** (for security-sensitive code)

---

**Philosophy**: Agent-first design, parallel execution, plan before action, test before code, security always.
