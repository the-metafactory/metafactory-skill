# metafactory-skill-package-builder

**PackageBuilder** — the canonical Claude Code skill for building conformant, arc-installable packages for the metafactory ecosystem.

This repo is the single source of truth for PackageBuilder. It absorbs the canonical 985-line skill previously carried in `arc/skill/` (WS3 of the skill-estate migration epic, [the-metafactory/arc#316](https://github.com/the-metafactory/arc/issues/316)); the arc-side copy is removed in a paired atomic release so no component keeps two sources of truth (cortex ADR-0024 D2).

The skill encodes the conventions, governance rules, and quality requirements that otherwise live scattered across dozens of CLAUDE.md files, design decisions, and SOPs — so contributors and agents can build packages at scale without re-learning tacit knowledge each time.

## What it does

PackageBuilder guides you through:

- Scaffolding a new arc-installable package (skill, tool, agent, prompt, component, pipeline, or action)
- Authoring a valid `arc-manifest.yaml` with correct capabilities, tier, and namespace
- Meeting blueprint tracking, compass governance, content-filter safety, and test-rig verification requirements
- Composing blueprints and authoring persona-driven agents
- Preparing a package for submission to the metafactory registry with PR-quality standards

## When to use

Activate via any of these triggers: `build package`, `create package`, `metafactory package`, `author-builder`, `package conventions`, `author persona agent`, `compose blueprints`, `publish bundle`.

Use it when creating a new arc-installable package, preparing one for registry submission, reviewing whether a package meets ecosystem conventions, or onboarding as an Author-Builder. Do **not** use it for work on the metafactory registry itself, or on arc/compass/grove infrastructure repos — those have their own conventions.

## Workflows

| Workflow | Purpose | File |
|----------|---------|------|
| **CreatePackage** | Scaffold a new arc-installable package | `skill/Workflows/CreatePackage.md` |
| **SubmitPackage** | Prepare and submit a package for registry review | `skill/Workflows/SubmitPackage.md` |
| **PublishBundle** | Publish a bundle-style repo | `skill/Workflows/PublishBundle.md` |
| **AuthorPersonaAgent** | Author a persona-driven agent | `skill/Workflows/AuthorPersonaAgent.md` |

## Structure

```
metafactory-skill-package-builder/
  arc-manifest.yaml              # arc/v1 package contract
  README.md
  skill/
    SKILL.md                     # skill entry point + convention reference
    Workflows/                   # one .md per discrete operation
```

## Installation

```bash
arc install metafactory-skill-package-builder
```

The skill installs to `~/.claude/skills/PackageBuilder/` and activates automatically in Claude Code when a trigger phrase is matched.

## Manifest

- **Name**: `package-builder`
- **Version**: `0.1.0`
- **Type**: `skill`
- **Tier**: `official`
- **Namespace**: `@metafactory`
- **License**: Apache-2.0

## Author

Andreas Aastroem — [@mellanon](https://github.com/mellanon)

## Inspiration

Modeled on the HuggingFace transformers-to-mlx methodology: codify tacit knowledge so quality scales with contributions, not review capacity.
