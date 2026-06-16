# Agent Skills Marketplace

A curated registry of Agent Skills authored by [@shivdeepak](https://github.com/shivdeepak), compatible with **Claude Code**, **Cursor**, **Codex CLI**, Claude Web, and Claude Cowork.

The registry is [`catalog.json`](./catalog.json). Skills live in their own repos — this repo is index-only.

---

## Adding this marketplace

### Claude Code

```
/plugin marketplace add shivdeepak/marketplace
```

This registers the catalog with Claude Code. Run `/plugin` and go to the **Discover** tab to browse available skills, then install one:

```
/plugin install <skill-name>@shivdeepak-marketplace
```

After installing, run `/reload-plugins` and invoke the skill with `/<skill-name>`.

### Cursor

**Teams / Enterprise** — go to **Dashboard → Settings → Plugins**, click **Import** under Team Marketplaces, and paste `https://github.com/shivdeepak/marketplace`.

Individual skills that are published to the Cursor Marketplace can also be installed directly inside the editor with `/add-plugin`.

### Codex CLI

```bash
npx codex-marketplace add shivdeepak/marketplace --plugins --project
```

Or launch `codex` and type `/plugins` to browse and install interactively.

---

## Skills

### [speakeasy](https://github.com/shivdeepak/speakeasy) `v1.0.2`
Coaches spoken English after each reply on voice input. Flags grammar, word choice, and idiom errors; stays silent when clean.

### [self-documenting](https://github.com/shivdeepak/self-documenting) `v1.1.2`
Keeps a project self-documenting by updating docs alongside code — triggered by any change that adds, removes, or alters behavior, config, commands, APIs, or project structure.

### [knowledge-base-builder](https://github.com/shivdeepak/knowledge-base-builder) `v1.1.1`
Turns a folder of notes, docs, or plans into an LLM-navigable knowledge base using `index.md` files and per-note summaries. No vector DB required.

### [skillship](https://github.com/shivdeepak/skillship) `v1.2.2`
Drives the `skillship` CLI to scaffold, validate, package, and publish Agent Skills across platforms.

---

## Installing a skill

### Claude Code

```bash
npx skillship install shivdeepak/<skill-name> -a claude-code
```

Or using the universal installer:

```bash
npx skills add shivdeepak/<skill-name>
```

### Cursor

```bash
npx skillship install shivdeepak/<skill-name> -a cursor
```

### Codex CLI

Add the skill's instruction block to your project's `AGENTS.md`:

```bash
npx skills add shivdeepak/<skill-name> --agent codex
```

If the Codex adapter is not available via `npx skills`, grab the skill content from the repo's `SKILL.md` and append it to your `AGENTS.md` manually, or use the `snippets/agents-md.md` file if present in the skill repo.

### Claude Web / Cowork (upload)

```bash
npx skillship package shivdeepak/<skill-name>
# → dist/<skill-name>.skill
```

Upload `dist/<skill-name>.skill` via Settings → Capabilities → Upload skill.

---

## Adding a skill to this registry

Edit [`catalog.json`](./catalog.json) — add an entry following the schema in [`catalog-schema.json`](./catalog-schema.json). Keep `description` under 200 characters (Claude Web upload limit).
