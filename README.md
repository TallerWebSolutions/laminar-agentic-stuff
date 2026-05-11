# Laminar agentic assets

This repository holds reusable **agent skills**, **plugins**, and related playbooks for AI-assisted work around **Laminar**: a platform for managing demands with kanban boards, story maps, and the broader context of digital products under development.

## What's inside

- **`skills/`** — Cursor-style skills (`SKILL.md` entrypoints, optional `references/`).
- **`plugins/`** — Reserved for Laminar-related plugins (MCP extensions, integrations, or tooling); add packages or manifests here as they land.

## Repository layout

```
skills/
  <skill-name>/
    SKILL.md
    references/
      supporting-doc.md

plugins/
  (future plugin packages or manifests)
```

Each skill folder includes a `SKILL.md` entrypoint and optional supporting documents under `references/`.

## Contributing

Add new skills under `skills/<name>/`. Follow the same frontmatter + markdown pattern as existing skills. When referencing remote MCP behavior, treat the live server's tool definitions as authoritative.

## License

MIT — see [LICENSE](./LICENSE).
