# Agent Skills Marketplace

This repo is a registry of Agent Skills by @shivdeepak. The authoritative index is `catalog.json`.

## Reading the catalog

```
catalog.json          — machine-readable skill registry (name, description, repo, version, platforms)
catalog-schema.json   — JSON Schema for catalog.json
README.md             — human-readable landing page with install commands
```

Each skill lives in its own repo listed under `"repo"` in `catalog.json`. This repo contains no skill files.

## Available skills

| name | description | repo |
|------|-------------|------|
| speakeasy | Coaches spoken English on voice input | https://github.com/shivdeepak/speakeasy |
| self-documenting | Updates docs alongside code changes | https://github.com/shivdeepak/self-documenting |
| knowledge-base-builder | Turns notes into an LLM-navigable knowledge base | https://github.com/shivdeepak/knowledge-base-builder |
| skillship | Scaffold, validate, and publish Agent Skills | https://github.com/shivdeepak/skillship |

## Installing a skill in this session (Codex CLI)

To install a skill for the current project, append its `SKILL.md` content to the project's `AGENTS.md`:

```bash
# Via npx skills (if Codex adapter is registered)
npx skills add shivdeepak/<skill-name> --agent codex

# Or fetch SKILL.md directly and append
curl -sL https://raw.githubusercontent.com/shivdeepak/<skill-name>/main/<skill-name>/SKILL.md >> AGENTS.md
```

## Adding a new skill to the registry

1. Edit `catalog.json` — add an entry with `name`, `description` (≤200 chars), `repo`, `version`, `license`, `platforms`, and `tags`.
2. Update the table in this file.
3. Commit with `feat: add <skill-name> to catalog`.
