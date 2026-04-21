# my-agentic-skills

sassman's opinionated agentic skills and agents — personal tools for git workflows and ratatui TUI development. Each plugin can be installed independently.

## Install

First, add the marketplace:

```bash
claude plugin marketplace add sassman/my-agentic-skills
```

Then install the plugins you want:

```bash
# Install everything
claude plugin install sas@my-agentic-skills
claude plugin install rust-specialists@my-agentic-skills

# Or just the agent
claude plugin install rust-specialists@my-agentic-skills

# Or just the commands
claude plugin install sas@my-agentic-skills
```

## Available plugins

### `sas` — Opinionated git commands

```bash
claude plugin install sas@my-agentic-skills
```

Commit messages and PR descriptions that are non-verbose, useful, and crisp. Very opinionated.

| Command | Invocation | Description |
|---|---|---|
| **git-commit** | `/sas:git-commit` | Concise, meaningful commit messages |
| **pr-create** | `/sas:pr-create` | Crisp PR titles and descriptions — no fluff |

### `rust-specialists` — Rust framework specialist agents

```bash
claude plugin install rust-specialists@my-agentic-skills
```

Strictly enforces the Pure/SideEffect message separation pattern from [ratatui with the elm architecture — handling side effects](https://d34dl0ck.me/ratatui-elm-architecture-side-effects/). Activates automatically when Claude detects you're working on ratatui TUI code — no slash command needed.

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
