# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the official Claude Skills repository maintained by Anthropic. It contains skill definitions that teach Claude how to complete specific tasks in a repeatable way.

## Directory Structure

- `skills/` - Collection of skill folders, each containing a `SKILL.md` with instructions and metadata
- `spec/` - Agent Skills specification
- `template/` - Template for creating new skills

## Skill Categories

The skills are organized into categories within `skills/`:

- **Creative & Design**: algorithmic-art, canvas-design, brand-guidelines, frontend-design, theme-factory
- **Development & Technical**: claude-api, mcp-builder, webapp-testing, web-artifacts-builder, skill-creator
- **Enterprise & Communication**: internal-comms, doc-coauthoring, slack-gif-creator
- **Document Skills**: docx, pdf, pptx, xlsx (source-available, not open source)

## Creating a New Skill

A skill is a folder containing a `SKILL.md` file with YAML frontmatter:

```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Instructions that Claude will follow when this skill is active]
```

Required frontmatter fields:
- `name` - Unique identifier (lowercase, hyphens for spaces)
- `description` - When to use this skill and what it does

## Using This Repository with Claude Code

Register as a marketplace plugin:
```
/plugin marketplace add anthropics/skills
```

Install specific skill sets:
```
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

## Key Files

- `README.md` - Main project documentation
- `THIRD_PARTY_NOTICES.md` - Third-party licenses
- `.claude-plugin/marketplace.json` - Plugin marketplace configuration
