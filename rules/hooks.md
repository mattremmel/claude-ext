# Hooks System

## Hook Types

- **PreToolUse**: Before tool execution (validation, parameter modification)
- **PostToolUse**: After tool execution (auto-format, checks)
- **Stop**: When session ends (final verification)

## Current Hooks (in ~/.claude/settings.json)

### PreToolUse
- **tmux reminder**: Suggests tmux for long-running commands (package managers, build tools)
- **git push review**: Opens editor for review before push
- **doc blocker**: Blocks creation of unnecessary .md/.txt files

### PostToolUse
- **PR creation**: Logs PR URL and CI status
- **Auto-format**: Runs language-appropriate formatter after edit
- **Type check**: Runs type checker after editing typed files
- **Debug statement warning**: Warns about debug statements in edited files

### Stop
- **Debug statement audit**: Checks modified files for debug statements before session ends

## Auto-Accept Permissions

Use with caution:
- Enable for trusted, well-defined plans
- Disable for exploratory work
- Never use dangerously-skip-permissions flag
- Configure `allowedTools` in `~/.claude.json` instead

## TodoWrite Best Practices

Use TodoWrite tool to:
- Track progress on multi-step tasks
- Verify understanding of instructions
- Enable real-time steering
- Show granular implementation steps

Todo list reveals:
- Out of order steps
- Missing items
- Extra unnecessary items
- Wrong granularity
- Misinterpreted requirements

---

## Language-Specific Examples

- [JavaScript/TypeScript](languages/javascript/hooks.md) - npm/pnpm/yarn, Prettier, tsc, console.log detection
- [Python](languages/python/hooks.md) - pip/poetry/uv, Black/Ruff, mypy, print detection
- [Go](languages/go/hooks.md) - go mod/build, gofmt, golangci-lint, fmt.Println detection
- [Rust](languages/rust/hooks.md) - cargo commands, rustfmt, clippy, println! detection
