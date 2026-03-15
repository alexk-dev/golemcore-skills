# AGENTS.md — Authoring Skills for `golemcore-skills`

This file tells AI agents and contributors how to add or update skill artifacts in this repository.

`golemcore-skills` is a **registry repository**. Its published runtime unit is still a `SKILL.md`, but the repository contract also requires stable maintainer namespaces, artifact manifests, and predictable paths.

Use this guide when you are asked to create:
- a new maintainer namespace
- a standalone skill artifact
- a pack containing multiple skills
- repository documentation related to skill authoring

All repository content, metadata, commit messages, PR titles, and PR descriptions must be written in **English**.

---

## 1. Read These Files First

Before changing anything, read the repository contract:

1. `README.md`
2. `CONTRIBUTING.md`
3. `docs/REPOSITORY_STRUCTURE.md`

If you are authoring or reviewing `SKILL.md` content, also align it with the current Golemcore Bot skill runtime contract from `golemcore-bot`, especially:

- `docs/SKILLS.md`
- the `Skill` model
- the `SkillService` loader/parser
- the `SkillVariableResolver`

That runtime contract matters because this repository publishes skills meant to be consumable by Golemcore tooling.

---

## 2. Core Repository Model

Use these terms precisely:

- **maintainer**: namespace owner
- **artifact**: install/update unit
- **skill**: runtime prompt definition in `SKILL.md`

Canonical references:

- Maintainer: `<maintainer-slug>`
- Standalone artifact: `<maintainer-slug>/<artifact-id>`
- Pack skill: `<maintainer-slug>/<artifact-id>/<skill-id>`

### Required layouts

### Standalone skill

```text
registry/<maintainer-slug>/
  maintainer.yaml
  <artifact-id>/
    artifact.yaml
    SKILL.md
```

### Pack

```text
registry/<maintainer-slug>/
  maintainer.yaml
  <artifact-id>/
    artifact.yaml
    skills/
      <skill-id>/SKILL.md
      <skill-id>/SKILL.md
```

Rules:
- one maintainer directory per namespace
- one artifact per directory
- one `SKILL.md` at artifact root for standalone artifacts
- one `skills/<skill-id>/SKILL.md` per pack skill
- no hidden extra skills outside the declared manifest

---

## 3. Naming and Identity Rules

Use stable lowercase slugs only:

```text
[a-z0-9][a-z0-9-]*
```

Apply this to:
- maintainer ids
- artifact ids
- pack-local skill ids
- skill `name` values

Never use:
- spaces
- underscores
- uppercase letters
- display names as runtime ids

Identity rules:
- do not rename published maintainers casually
- do not reuse an existing artifact id for unrelated content
- do not rename pack-local skill ids without an explicit migration reason
- do not silently change a skill’s runtime `name` once published

---

## 4. Files You Must Create or Update

### `maintainer.yaml`

Path:

```text
registry/<maintainer-slug>/maintainer.yaml
```

Minimum structure:

```yaml
schema: v1
id: golemcore
display_name: Golemcore
```

Recommended optional fields:

```yaml
github: golemcore
contact: team@golemcore.dev
```

Rules:
- `id` must equal the directory name
- display name may change over time
- slug should remain stable

### `artifact.yaml`

Every artifact directory must contain exactly one `artifact.yaml`.

#### Standalone artifact example

```yaml
schema: v1
type: skill
maintainer: golemcore
id: code-reviewer
version: 1.0.0
title: Code Reviewer
description: Review code changes for correctness, regressions, security, and missing tests.
```

#### Pack artifact example

```yaml
schema: v1
type: pack
maintainer: golemcore
id: devops-pack
version: 1.2.0
title: DevOps Pack
description: Operational skills for delivery, incidents, and change review.
skills:
  - id: deploy-review
    path: skills/deploy-review/SKILL.md
  - id: incident-triage
    path: skills/incident-triage/SKILL.md
```

Rules:
- `maintainer` must match the parent namespace
- `id` must match the artifact directory name
- `type` must be `skill` or `pack`
- every pack skill declared in `artifact.yaml` must exist on disk
- every content change intended for publication should bump `version`

Recommended future-compatible optional fields:

```yaml
compatibility:
  golemcore: ">=0.15.0 <0.16.0"
source:
  repository: https://github.com/alexk-dev/golemcore-skills
tags:
  - review
  - quality
```

---

## 5. `SKILL.md` Authoring Contract

A repository artifact is only useful if its `SKILL.md` matches the runtime expectations of `golemcore-bot`.

Each `SKILL.md` must contain:
1. YAML frontmatter between `---` delimiters
2. a Markdown instruction body
3. self-contained, reviewable content

### Minimal compatible shape

```markdown
---
name: code-reviewer
description: Review code changes with a findings-first senior engineer mindset.
model_tier: smart
---

Review the current changes as a senior engineer.
Prioritize correctness, regressions, security, and missing test coverage.
```

---

## 6. Metadata Fields Supported by `golemcore-bot`

Below is the practical compatibility contract to follow when publishing skills for Golemcore Bot.

### Required in practice

#### `name`
- Runtime identity of the skill.
- For standalone artifacts, keep it aligned with the artifact id.
- For pack skills, keep it aligned with the pack-local `skill-id`.
- Even though the bot can derive a name from the directory if omitted, **always set it explicitly in this repository**.

#### `description`
- Short user-facing summary.
- Keep it concise and specific.
- It should still make sense when shown in a list of available skills.

### Recommended

#### `model_tier`
Valid values in current Golemcore Bot docs:
- `balanced`
- `smart`
- `coding`
- `deep`

Use it when the skill clearly benefits from a particular model class.

Examples:
- `balanced` for simple conversational or lightweight routing skills
- `smart` for general analysis and planning
- `coding` for implementation-heavy tasks
- `deep` for research-intensive or expert reasoning tasks

### Requirements metadata

```yaml
requires:
  env:
    - GITHUB_TOKEN
  binary:
    - node
    - npx
  skills:
    - base-assistant
```

Use cases:
- `requires.env`: required external credentials or environment configuration
- `requires.binary`: required local executables
- `requires.skills`: related skill dependencies for future/consumer tooling

Important nuance:
- In current `golemcore-bot`, availability is checked from requirements and missing required variables.
- `requires.env` is actively meaningful.
- `requires.binary` is declarative and may be enforced only partially depending on runtime/tooling.
- `requires.skills` may be useful as metadata, but do not rely on it as the only safety mechanism unless your target consumer explicitly enforces it.

### Variable metadata

```yaml
vars:
  API_KEY:
    description: API key for the external service
    required: true
    secret: true
  ENDPOINT: https://api.example.com
  TIMEOUT:
    default: "30"
```

Supported patterns:
- full object definition with `description`, `default`, `required`, `secret`
- string shorthand where the string becomes the default value
- empty declaration when only the variable name is needed

Important runtime behavior:
- unresolved `required: true` variables make the skill unavailable
- `secret: true` values are masked in logs
- content placeholders use `{{VAR_NAME}}`

Current `golemcore-bot` variable resolution priority:
1. `skills/<skill-name>/vars.json`
2. `variables.json` → skill section
3. `variables.json` → `_global`
4. environment variables
5. `default` from frontmatter

Guidance:
- declare secrets as variables, never hardcode them in the prompt body
- prefer explicit `description` for non-obvious variables
- use defaults only for non-sensitive, predictable values

### MCP metadata

```yaml
mcp:
  command: npx -y @modelcontextprotocol/server-github
  env:
    GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_TOKEN}
  startup_timeout: 45
  idle_timeout: 10
```

Use MCP when the skill needs external tools exposed through a Model Context Protocol server.

Rules and nuances:
- `command` is required if `mcp` is present
- `env` values may reference `${VAR_NAME}` placeholders
- current Golemcore Bot resolves `${VAR_NAME}` from resolved skill variables first, then system environment
- `startup_timeout` is in seconds
- `idle_timeout` is in minutes
- if the skill needs `npx`, `node`, or specific credentials, declare them under `requires`

### Pipeline metadata

```yaml
next_skill: review-summary
conditional_next_skills:
  critical: security-audit
  tests: test-generator
```

Rules and nuances:
- `next_skill` and `conditional_next_skills` reference **runtime skill names**, not artifact paths
- the targets must be stable and installed together in the expected environment
- for pack skills, choose pack-local ids that are distinctive enough for runtimes that index skills globally by `name`
- do not create transition chains that depend on unpublished or ambiguous names

### About `available`

You may see `available: true` in existing examples.

Important nuance:
- current `golemcore-bot` computes runtime availability from requirements and missing required variables
- a static `available:` value should be treated as informational at most
- do not rely on `available:` to force a skill to load or bypass unmet requirements

Preferred guidance:
- omit `available` unless the target registry convention explicitly wants it for display purposes
- if included for consistency with existing repository examples, do not treat it as the source of truth

---

## 7. How to Write the Instruction Body

The body should be executable guidance for the model, not marketing copy.

Write bodies that are:
- specific about the task
- explicit about priorities
- explicit about inputs, outputs, and constraints
- safe around destructive actions
- self-contained enough to review in isolation

Prefer:
- concrete workflows
- ordered steps
- review criteria
- output contracts
- decision rules

Avoid:
- vague persona-only prompts
- hardcoded secrets
- repository-specific assumptions without naming them
- unsupported metadata inventions when normal Markdown instructions would work
- style-only verbosity with no operational guidance

### Good body structure

1. One-line role statement
2. Workflow / ordered process
3. What to optimize for
4. Edge cases / failure handling
5. Output contract
6. Safety rules for destructive actions or external systems

### Safety guidance

If a skill can trigger side effects, say so explicitly.

Examples:
- require confirmation before destructive actions
- avoid deleting or closing resources without explicit instruction
- distinguish read-only investigation from write actions
- instruct the model to call out uncertainty instead of guessing

---

## 8. Templates

### Template: standalone skill artifact

`artifact.yaml`

```yaml
schema: v1
type: skill
maintainer: <maintainer-slug>
id: <artifact-id>
version: 1.0.0
title: <Human Title>
description: <One-sentence artifact description>
```

`SKILL.md`

```markdown
---
name: <artifact-id>
description: <Short runtime description>
model_tier: smart
requires:
  env:
    - OPTIONAL_ENV_VAR
  binary:
    - OPTIONAL_BINARY
vars:
  OPTIONAL_ENV_VAR:
    description: Credential for the external system
    required: true
    secret: true
mcp:
  command: <optional MCP command>
  env:
    OPTIONAL_ENV_VAR_ALIAS: ${OPTIONAL_ENV_VAR}
  startup_timeout: 30
  idle_timeout: 5
---

You are <role>.

## Workflow

1. Inspect the user request and identify the real goal.
2. Gather only the missing context required to act correctly.
3. Execute the task with the available tools and constraints.
4. Surface blockers, uncertainty, or missing permissions explicitly.

## Priorities

- correctness
- safety
- explicit assumptions
- concise, actionable output

## Output Contract

- explain the result
- list important risks or follow-ups
- say explicitly when verification was not possible
```

### Template: pack artifact

`artifact.yaml`

```yaml
schema: v1
type: pack
maintainer: <maintainer-slug>
id: <pack-id>
version: 1.0.0
title: <Pack Title>
description: <One-sentence pack description>
skills:
  - id: <skill-id-one>
    path: skills/<skill-id-one>/SKILL.md
  - id: <skill-id-two>
    path: skills/<skill-id-two>/SKILL.md
```

`skills/<skill-id-one>/SKILL.md`

```markdown
---
name: <skill-id-one>
description: <Short runtime description>
model_tier: balanced
next_skill: <skill-id-two>
---

<Instructions>
```

`skills/<skill-id-two>/SKILL.md`

```markdown
---
name: <skill-id-two>
description: <Short runtime description>
model_tier: coding
---

<Instructions>
```

---

## 9. Review Checklist Before Committing

Validate all of the following:

### Repository structure
- the path matches `docs/REPOSITORY_STRUCTURE.md`
- the maintainer directory exists and is correct
- the artifact id matches the directory name
- the pack manifest matches the actual files on disk

### Metadata quality
- all text is in English
- all ids are stable lowercase slugs
- `name` is explicit and aligned with the artifact id or pack-local skill id
- `description` is specific and useful
- `model_tier` uses a supported value if present
- secrets are declared via `vars`, not hardcoded
- transition targets refer to runtime names that are expected to exist
- MCP metadata declares real dependencies and uses variable placeholders safely

### Prompt quality
- body is task-oriented, not just descriptive
- output expectations are explicit
- destructive actions require confirmation when appropriate
- instructions are self-contained and reviewable

### Publication hygiene
- `artifact.yaml` version was bumped for publishable content changes
- published identifiers were not changed accidentally
- no unrelated files were modified

---

## 10. Branch, Commit, and PR Rules

### Branches
Use a descriptive topic branch, for example:
- `feat/add-deployment-pack`
- `feat/add-code-review-skill`
- `docs/add-skill-authoring-guide`
- `fix/update-pack-metadata`

### Commits
Use Conventional Commits.

Good examples:
- `feat(skill): add golemcore/code-reviewer`
- `feat(pack): add golemcore/devops-pack`
- `feat(maintainer): add acme namespace metadata`
- `docs: add AGENTS guide for skill authors`
- `fix(skill): correct metadata for golemcore/code-reviewer`

### Pull requests
Direct commits to `main` are prohibited. Open a PR from a topic branch.

PR titles are validated in CI and must follow Conventional Commits.

Every PR should explain:
- what changed
- which maintainer namespace is affected
- whether this adds a standalone skill, a pack, or repository-level documentation
- whether any canonical artifact references changed
- whether consumers need metadata or install-surface updates

When the change touches skill metadata compatibility, mention that explicitly in the PR summary.

---

## 11. Preferred Agent Behavior for This Repository

When asked to add a new skill, do not stop at drafting prompt text only.

Produce the full artifact surface:
- `maintainer.yaml` if needed
- `artifact.yaml`
- `SKILL.md`
- pack `skills/` entries if applicable
- any necessary repository docs updates if the repository contract changed

When asked to review an existing skill, check both:
1. repository structure correctness
2. Golemcore Bot runtime compatibility

If those two contracts conflict, keep the repository structure valid and call out the compatibility issue explicitly in the PR or review.

---

## 12. Short Decision Rules

If unsure, default to these rules:
- explicit metadata is better than implicit metadata
- stable ids are more important than pretty names
- runtime compatibility is more important than decorative fields
- secrets belong in variables, not in prompt text
- pipeline targets must use actual runtime skill names
- English only
- one artifact per directory
- one clear PR with a detailed summary
