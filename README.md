# metafactory-skill

**PackageBuilder** — a Claude Code skill for building conformant, arc-installable packages for the metafactory ecosystem.

This skill encodes the conventions, governance rules, and quality requirements that live across dozens of CLAUDE.md files, design decisions, and SOPs in the metafactory ecosystem. It exists so contributors and agents can build packages at scale without re-learning tacit knowledge each time.

## What It Does

PackageBuilder guides you through:

- Scaffolding a new arc-installable package (skill, tool, agent, prompt, component, pipeline, or action)
- Authoring a valid `arc-manifest.yaml` with correct capabilities, tier, and namespace
- Meeting blueprint tracking, compass governance, content-filter safety, and test-rig verification requirements
- Preparing a package for submission to the metafactory registry with PR-quality standards

## When to Use

Activate via any of these triggers:

- `build package`
- `create package`
- `metafactory package`
- `author-builder`
- `package conventions`

Use it when:

- Creating a new arc-installable package
- Preparing a package for submission to the registry
- Reviewing whether a package meets ecosystem conventions
- Onboarding as an Author-Builder

Do **not** use it for work on the metafactory registry itself, or on arc/compass/grove infrastructure repos — those have their own conventions.

## Workflows

| Workflow | Trigger | File |
|----------|---------|------|
| **CreatePackage** | "build package", "create package", "author-builder onramp" | `skill/Workflows/CreatePackage.md` |
| **SubmitPackage** | "submit package", "package review", "publish package" | `skill/Workflows/SubmitPackage.md` |

## Structure

```
metafactory-skill/
  arc-manifest.yaml              # arc package contract
  skill/
    SKILL.md                     # skill entry point + convention reference
    Workflows/
      CreatePackage.md           # scaffolding workflow
      SubmitPackage.md           # submission workflow
```

## Installation

```bash
arc install metafactory-skill
```

The skill installs to `~/.claude/skills/PackageBuilder/` and activates automatically in Claude Code when a trigger phrase is matched.

## Manifest

- **Name**: `metafactory-skill`
- **Version**: `0.1.0`
- **Type**: `skill`
- **Namespace**: `the-metafactory`
- **License**: Apache-2.0

## Author

Jens-Christian Fischer — [@jcfischer](https://github.com/jcfischer)

## Inspiration

Modeled on the HuggingFace transformers-to-mlx methodology: codify tacit knowledge so quality scales with contributions, not review capacity.
