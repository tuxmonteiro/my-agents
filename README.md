# my-agents

A collection of custom [OpenCode](https://opencode.com) agent definitions. Each agent is a self-contained persona with a specific purpose, toolset, and set of behavioral rules — packaged as a single Markdown file with YAML frontmatter.

## What is an OpenCode Agent?

An OpenCode agent is a prompt file that defines a custom AI persona. Unlike generic instructions, agents can specify:

- **A name and description** for the persona
- **A mode** (`primary`, `subagent`, `agentic`) controlling how OpenCode routes requests to it
- **Tool permissions** — fine-grained control over which tools the agent may use (e.g., `bash`, `file_editor`, `read_file`, `write_file`)
- **Behavioral rules** written in Markdown that shape how the agent thinks and acts

Agents live as `.md` files in an `agents/` directory (or any directory configured in your `opencode.json`). OpenCode loads them and exposes them for invocation.

## Agents Overview

| Agent | Purpose | Mode | Tools |
|-------|---------|------|-------|
| [restricted-build](#restricted-build) | Full-featured build agent strictly isolated to the current directory and `/tmp` | `primary` | bash, file_editor, read_file, write_file |

---

## restricted-build

**Description:** Full-featured build agent strictly isolated to the current directory and `/tmp`.

**Mode:** `primary` — the default agent used for general coding tasks.

**Toolset:**
- `bash` — run commands, build, test, compile
- `file_editor` — edit existing files
- `read_file` — read file contents
- `write_file` — create and overwrite files

### Directory Boundary Rules

This agent enforces **strict directory isolation** as a mandatory behavioral rule:

1. **Permitted Change Locations:**
   - Current Working Directory (`.`) and its subdirectories
   - Temporary Directory (`/tmp`) and its subdirectories

2. **Absolute Restrictions:**
   - **DO NOT** create, edit, delete, or overwrite files outside of `.` or `/tmp`
   - **DO NOT** run bash commands that alter files, install packages globally, or mutate state outside `.` or `/tmp` (e.g., `~`, `/etc`, `/usr`, `/var`, `..`)

3. **No-Prompt Enforcement:**
   - **NEVER** ask the user for permission to access, read, or modify directories outside of `.` and `/tmp`
   - If a requested action requires modifying files outside these allowed boundaries, **silently limit the scope** to `.` or `/tmp`, or decline that specific off-boundary write without prompting for elevated path access

### When to use

- When you need to compile, test, and edit files but want to guarantee no changes leak outside the project workspace
- When working in a shared or production environment where global state mutations must be prevented
- As a safe default agent for build-heavy tasks with strong containment guarantees

---

## Repository Structure

```
my-agents/
└── restricted-build.md
```

Each agent is a single Markdown file with YAML frontmatter:

```markdown
---
name: <agent-name>
description: <one-line summary>
mode: <primary | subagent | agentic>
tools:
  bash: true
  file_editor: true
  read_file: true
  write_file: true
---

<agent prompt / behavioral rules>
```

## Usage

This repository is designed to be cloned or symlinked into your OpenCode agents directory:

```bash
# Clone alongside other agent collections
git clone git@github.com:tuxmonteiro/my-agents.git ~/.config/opencode/agents/

# Or add as a separate agents source
git clone git@github.com:tuxmonteiro/my-agents.git /path/to/project/opencode-agents/
```

Then reference the agents in your `opencode.json`:

```jsonc
{
  "agents": [
    "opencode-agents/restricted-build"
  ]
}
```

Or invoke directly by name from the CLI:

```bash
opencode --agent restricted-build
```

## Related

- **[my-skills](https://github.com/tuxmonteiro/my-skills)** — A companion repository containing specialized OpenCode skills (structured workflows for tasks like TDD, git commits, planning, ADRs, and more).
