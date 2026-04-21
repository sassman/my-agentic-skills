# my-agentic-skills

Personal Claude Code marketplace with custom commands and specialized agents. Each plugin can be installed independently.

## Install

First, add the marketplace:

```bash
claude plugin marketplace add sassman/my-agentic-skills
```

Then install the plugins you want:

```bash
# Install everything
claude plugin install sas@my-agentic-skills
claude plugin install ratatui-elm-architect@my-agentic-skills

# Or just the agent
claude plugin install ratatui-elm-architect@my-agentic-skills

# Or just the commands
claude plugin install sas@my-agentic-skills
```

## Available plugins

### `sas` — Custom commands

```bash
claude plugin install sas@my-agentic-skills
```

| Command | Invocation | Description |
|---|---|---|
| **git-commit** | `/sas:git-commit` | Create a git commit with a meaningful message |
| **pr-create** | `/sas:pr-create` | Create a pull request with a meaningful title and description |

### `ratatui-elm-architect` — Ratatui + Elm Architecture agent

```bash
claude plugin install ratatui-elm-architect@my-agentic-skills
```

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
