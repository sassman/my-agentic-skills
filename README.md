# my-agentic-skills

Personal Claude Code plugin with custom commands and specialized agents.

## Install

```bash
claude plugin add sassman/my-agentic-skills
```

## What's included

### Commands

| Command | Invocation | Description |
|---|---|---|
| **git-commit** | `/my:git-commit` | Create a git commit with a meaningful message |
| **pr-create** | `/my:pr-create` | Create a pull request with a meaningful title and description |

### Agents

| Agent | Activation | Description |
|---|---|---|
| **ratatui-elm-architect** | Automatic | Expert in ratatui TUI apps with the Elm Architecture — enforces pure update functions, Pure/SideEffect message separation, and testable state transitions |

## Agent details

### ratatui-elm-architect

This agent activates automatically when Claude detects you're working on ratatui TUI code. No slash command needed — just work on your ratatui project and Claude will use the agent's expertise when relevant.

**What it enforces:**

- `Message::Pure` / `Message::SideEffect` split at the type level
- `update()` stays pure — no I/O, all decision logic lives here
- `handle_side_effect()` is a dumb I/O dispatcher — no decisions
- Intent -> SideEffect -> Result -> Decision -> Mutation message flow
- Testable state transitions without mocking

**What it flags:**

- I/O inside `update()`
- Decision logic inside `handle_side_effect()`
- Flat message enums without Pure/SideEffect separation
- Side effect functions that access the model

## License

MIT
