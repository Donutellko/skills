# donutellko-skills

Personal Claude Code skills marketplace. Contains custom skills and a curated subset of [anthropics/skills](https://github.com/anthropics/skills).

## RDPI Bootstrap

The main skill in this collection. Sets up the **RDPI (Research → Design → Plan → Implement)** development workflow for any project — splitting AI-assisted development into 4 phases, each running in a fresh context with structured artifact handoff.

Based on Dex Horthy's "Context Engineering" (QRSPI) and Dmitry Bereznitsky's "Process over Prompts."

- **Plugin:** [`plugins/rdpi-bootstrap`](./plugins/rdpi-bootstrap)
- **Skill:** [`plugins/rdpi-bootstrap/skills/rdpi-bootstrap`](./plugins/rdpi-bootstrap/skills/rdpi-bootstrap)

## Upstream Skills

Some skills are pulled from the official [anthropics/skills](https://github.com/anthropics/skills) repository and may be modified or updated locally. The upstream remote is tracked as `upstream`.

### Document Skills
Excel, Word, PowerPoint, and PDF processing — [`skills/docx`](./skills/docx), [`skills/pdf`](./skills/pdf), [`skills/pptx`](./skills/pptx), [`skills/xlsx`](./skills/xlsx).

### Example Skills
Algorithmic art, brand guidelines, canvas design, MCP builder, skill creator, web testing, and more — see [`skills/`](./skills/).

## Installation

Register this repository as a Claude Code Plugin marketplace:
```
/plugin marketplace add Donutellko/skills
```

Install a specific plugin:
```
/plugin install rdpi-bootstrap@donutellko-skills
/plugin install document-skills@donutellko-skills
/plugin install example-skills@donutellko-skills
```

## Structure

```
plugins/            — custom plugins & skills
  rdpi-bootstrap/   — RDPI workflow bootstrap skill
skills/             — upstream skills (from anthropics/skills)
spec/               — Agent Skills specification
template/           — skill template
```

## License

Custom skills (under `plugins/`) have their own licensing. Upstream skills retain their original licenses — see individual `LICENSE.txt` files and [THIRD_PARTY_NOTICES.md](./THIRD_PARTY_NOTICES.md).
