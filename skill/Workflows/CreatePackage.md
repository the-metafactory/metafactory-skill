# CreatePackage Workflow

> Scaffold and build a new arc-installable package from scratch, following all metafactory ecosystem conventions.

## Prerequisites

Before starting:
- [ ] Bun is installed (`bun --version`)
- [ ] Arc is installed (`arc --version`)
- [ ] The package name is unique (check `arc list` and the registry)
- [ ] The package type is decided (skill, tool, agent, prompt, component, pipeline, action)

---

## Steps

### 1. Determine the Repo Name and Artifact Class

**Action:** Ask the user what the package does, then apply the **class-choice rule** (SKILL.md §0) to pick the repo name — decide by the lead artifact:
- lead artifact is a `SKILL.md` → `metafactory-skill-<name>` (the default)
- lead artifact is a CLI, or multiple unrelated skills → `metafactory-bundle-<name>`
- inseparable from one app's runtime → `metafactory-<app>-skill-<name>`

**Rules:**
- `<name>` is lowercase-hyphenated (e.g. `code-review`, not `CodeReview`).
- The manifest `name:` is the repo name **minus its class prefix** (spec §4.2): repo `metafactory-skill-code-review` ⇒ `name: code-review`. The SKILL.md frontmatter `name:` is the PascalCase of that (`CodeReview`).
- `bundle` is a repo-name class, never the manifest `type:`.
- Register the new repo in `compass/ecosystem/repos.yaml` (with `visibility:`).

**Verify:** the repo name matches one grammar; `arc list` shows no name collision.

**Anti-pattern:** Do not default the manifest `type` to `skill`. Set it from the lead artifact (a CLI-led repo is `type: tool`). And never a prefix-less repo name for a new repo.

### 2. Scaffold the Directory Structure (spec §4)

**Action:** Create the bundle-style skill repo shape. Omit any directory you don't need — a procedure-only skill is just `arc-manifest.yaml` + `skill/`.

```bash
mkdir -p metafactory-skill-{name}/skill/Workflows
# add only what the package actually ships:
mkdir -p metafactory-skill-{name}/src         # tools the skill uses (bun CLIs)
mkdir -p metafactory-skill-{name}/commands     # slash-command .md files (provides.commands)
mkdir -p metafactory-skill-{name}/agents       # agent rule files (provides.agents)
mkdir -p metafactory-skill-{name}/hooks        # hook scripts (provides.hooks)
mkdir -p metafactory-skill-{name}/test
```

**Verify:** `ls -R metafactory-skill-{name}/` shows `arc-manifest.yaml` (next step) + `skill/SKILL.md` + `skill/Workflows/` at minimum.

### 3. Create arc-manifest.yaml

**Action:** Write `arc-manifest.yaml` at the package root using the arc/v1 schema.

Follow the strict arc/v1 contract (spec §4.1) exactly:
- `schema: arc/v1` (required literal; any legacy or absent schema is rejected by the validator).
- `name:` derives from the repo name minus its prefix (§4.2); `version: 0.1.0`.
- `type:` is the lead artifact's class (never `bundle`); `tier:` and a one-line `description:`.
- `license: Apache-2.0` (ecosystem default per DD-13, unless there's a specific reason for MIT).
- `author:` is a **singular map** `{ name, github }` — an `authors:` list is rejected.
- `namespace:` is OPTIONAL and **not identity** — if present it must be a DD-15 `@scope` (e.g. `@metafactory`), never a bare `the-metafactory`/username. Omit it unless you mean to declare a publish scope.
- `capabilities:` is a REQUIRED block with all four sub-blocks as **explicit empties** — `filesystem: { read: [], write: [] }`, `network: []`, `bash: { allowed: false }`, `secrets: []`. Network entries use the `{ host: <bare-host>, reason: <why> }` shape (never `{ domain, ... }` or a URL string).
- `bundle.exclude:` baseline `[vendor, MEMORY, node_modules, .git, Plans, test]`.

**Verify:** run `arc validate` in the repo root — it must exit 0 (see step 11). Reading the file back is not enough; the strict validator is the gate.

**Anti-pattern:** Do not copy capabilities from another package without checking if they apply — each package declares its *actual* surface (capability honesty). Do not omit any capabilities sub-block.

### 4. Create package.json

**Action:** Initialize bun project.

```bash
cd {name}
bun init -y
```

Edit `package.json` to set:
- `name`: matches arc-manifest name
- `scripts.test`: `"bun test"`
- `scripts.typecheck`: `"bunx tsc --noEmit"` (if using TypeScript)

**Verify:** `bun install` succeeds.

### 5. Create SKILL.md (Skill Type Only)

**Action:** Write `skill/SKILL.md` with YAML frontmatter and required sections.

Required in frontmatter:
- `name`: PascalCase skill name
- `description`: Multi-line, includes trigger phrases for activation matching
- `triggers`: Array of activation phrases

Required sections in body:
1. Title and overview (what and why)
2. When to Use / When NOT to Use
3. Workflow Routing Table (maps patterns to `Workflows/*.md` files)

**Verify:** YAML frontmatter parses correctly. All referenced workflow files exist (or are noted as TODO).

**Anti-pattern:** Do not write vague trigger phrases. "use tool" is too broad. "create changelog entry from git log" is specific.

### 6. Create CLI Entry Point (Tool Type Only)

**Action:** Write `src/cli.ts` with Commander.js structure.

```typescript
import { Command } from "commander";

const program = new Command()
  .name("{name}")
  .version("0.1.0")
  .description("{description}");

// Add subcommands here

program.parse();
```

**Verify:** `bun src/cli.ts --version` outputs `0.1.0`.

**Anti-pattern:** Do not use process.argv parsing directly. Use Commander.js for consistency with the ecosystem.

### 7. Create CLAUDE.md via agents-md.yaml

**Action:** Create `agents-md.yaml` at repo root and section files in `docs/agents-md/`.

```yaml
template: compass-standards
generate:
  - format: claude-md
repo_name: {name}
repo_description: "{description}"
deploy_command: "arc upgrade {name}"
version_source: arc-manifest.yaml
sections:
  - position: "after:description"
    file: docs/agents-md/architecture.md
  - position: "after:critical-rules"
    file: docs/agents-md/critical-rules.md
```

Create section files:
- `docs/agents-md/architecture.md`: Describe the package structure
- `docs/agents-md/critical-rules.md`: Add package-specific rules

Then generate: `arc upgrade compass`

**Verify:** CLAUDE.md exists at repo root with ecosystem-standard sections.

### 8. Create Blueprint (If Ecosystem Project)

**Action:** Create `blueprint.yaml` if the package will be registered in `compass/ecosystem/repos.yaml`.

```yaml
schema: blueprint/v1
repo: {short-name}
features:
  - id: {PREFIX}-001
    name: {first-feature}
    status: planned
    iteration: 1
    description: >
      {What the first feature delivers}
```

**Verify:** `blueprint lint` passes (no cycles, no dangling refs).

**Anti-pattern:** Do not create blueprint features that are too coarse. "Build the whole thing" is not a feature. Break into independently deliverable pieces.

### 9. Write Initial Tests

**Action:** Create at least one test file in `tests/`.

```typescript
import { describe, it, expect } from "bun:test";

describe("{name}", () => {
  it("manifest exists and parses", async () => {
    const manifest = await Bun.file("arc-manifest.yaml").text();
    expect(manifest).toContain("schema: arc/v1");
    expect(manifest).toContain(`name: {name}`);
  });
});
```

**Verify:** `bun test` passes.

### 10. Initialize Git and Create First Commit

**Action:** Initialize git, create .gitignore, make first commit.

```bash
git init
echo "node_modules/\n.env\nvendor/\nMEMORY/\n*.tar.gz" > .gitignore
git add .
git commit -m "chore: scaffold {name} package"
```

**Verify:** `git log` shows the initial commit. `git status` is clean.

### 11. Validate the manifest against the strict contract (the gate)

**Action:** Run arc's strict validator over the scaffolded repo — this is the acceptance gate; a scaffold that does not pass is not done.

```bash
arc validate            # in the repo root; exit 0 required
echo "exit=$?"
```

`arc validate` enforces spec §4.1/§4.2: `schema: arc/v1`, name-derivation, singular `author`, the full `capabilities` block with explicit empties, `{ host, reason }` network entries, `@scope` namespace grammar, and the SKILL.md frontmatter PascalCase-name rule. It rejects every legacy affordance (`pai/v1`, `authors:` lists, `{ domain, reason }`, missing capabilities). Wire the `metafactory-actions` `validate-manifest` reusable workflow as CI so the same gate runs on every PR.

**Verify:** `arc validate` prints `OK: …/arc-manifest.yaml is a valid arc/v1 manifest` and exits 0.

---

## Verification Checklist

After completing all steps:

- [ ] Repo named under the §3 grammar (`metafactory-skill-<name>` etc.); manifest `name` = repo name minus prefix
- [ ] `arc validate` exits 0 (the strict §4.1/§4.2 gate)
- [ ] Directory structure matches the spec §4 shape
- [ ] `arc-manifest.yaml` exists with all required fields
- [ ] `package.json` exists with correct name and test script
- [ ] `bun install` succeeds
- [ ] `bun test` passes
- [ ] SKILL.md exists with valid frontmatter (skill type only)
- [ ] CLI entry point runs (tool type only)
- [ ] `agents-md.yaml` and section files exist
- [ ] CLAUDE.md is generated
- [ ] Blueprint.yaml exists and lints (if ecosystem project)
- [ ] Git initialized with clean initial commit
- [ ] No secrets or .env files committed
- [ ] No em dashes in any documentation

## What NOT To Do

- Do not skip the arc-manifest.yaml. Every package needs one, even the simplest.
- Do not copy another package's manifest and change the name. Capabilities, dependencies, and structure differ per package.
- Do not use npm, yarn, or pnpm. Bun is the ecosystem standard.
- Do not commit on the main branch after initial scaffold. Use worktrees for all subsequent work.
- Do not assert "tests pass" without running `bun test`. Run it. Read the output.
