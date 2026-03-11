# Agent Instructions for Reqfleet Skills

This repository contains skill definitions that empower AI agents to interact with the [Reqfleet](https://reqfleet.com) platform. Use these instructions to maintain and extend the skills.

## Skill Structure

Each skill must be located in its own directory within the `skills/` folder and contain a `SKILL.md` file.

### SKILL.md Requirements

Every `SKILL.md` must start with YAML frontmatter containing `name` and `description`:

```markdown
---
name: reqfleet-basics
description: Guides the setup and usage of the Reqfleet CLI (rqt). Use when starting a new Reqfleet session.
---
# Skill Title

Skill instructions...
```

- **name**: Use lowercase, hyphenated names (e.g., `reqfleet-basics`).
- **description**: A single-line string clearly stating what the skill does and when an agent should use it.

## Workflow for Editing Skills

1.  **Research:** Read existing skills and the project `README.md` to understand current capabilities and conventions.
2.  **Edit/Create:** Modify existing skills or create new skill directories and `SKILL.md` files as needed.
3.  **Validate:** Always validate the skill format before finishing your task.

## Skill Validation

To ensure consistency and compatibility, all skills must be validated. Validation checks for:
- Correct YAML frontmatter format.
- Presence of required fields (`name` and `description`).
- Absence of `TODO` placeholders.

If you have access to a validation script (e.g., `validate_skill.cjs`), run it against the skill directory:

```bash
node path/to/validate_skill.cjs skills/<skill-directory>
```

If a validation tool is not available, perform a manual check against the requirements in this document.

## Conventions

- **Namespacing:** Prefix skill names with `reqfleet-` to avoid collisions with other skills or tools.
- **Security:** Never include API keys, secrets, or sensitive credentials in skill files.
- **Brevity:** Keep `SKILL.md` instructions concise, actionable, and focused on providing procedural knowledge that the model may not already possess.
- **Imperative Tone:** Use the imperative mood (e.g., "Run...", "Fetch...", "Create...") when writing instructions within `SKILL.md`.
