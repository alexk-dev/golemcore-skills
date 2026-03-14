# Repository Structure

This document defines the canonical layout for `golemcore-skills`.

It is intentionally strict because this repository must support:

- multiple maintainers
- standalone skills
- packs containing multiple skills
- future install tooling that can resolve artifact ownership and provenance without guessing

Consumer tooling may implement this contract incrementally, but repository content should still follow this structure from the start.

## Design Principles

- `maintainer` is the namespace boundary.
- `artifact` is the install and update unit.
- `skill` is the runtime execution unit.
- Paths should reveal ownership and artifact boundaries immediately.
- Identifiers must be stable over time.

## Core Terms

### Maintainer

A maintainer is the owner namespace for one or more published artifacts.

Canonical reference:

```text
<maintainer-slug>
```

Example:

```text
golemcore
```

### Artifact

An artifact is the published installable unit. An artifact can be either:

- a standalone skill
- a pack

Canonical artifact reference:

```text
<maintainer-slug>/<artifact-id>
```

Examples:

```text
golemcore/code-reviewer
golemcore/devops-pack
```

### Skill

A skill is the runtime definition stored in `SKILL.md`.

Canonical skill reference for a standalone artifact:

```text
<maintainer-slug>/<artifact-id>
```

Canonical skill reference inside a pack:

```text
<maintainer-slug>/<artifact-id>/<skill-id>
```

Example:

```text
golemcore/devops-pack/deploy-review
```

## Top-Level Layout

```text
.
├── CONTRIBUTING.md
├── README.md
├── docs/
│   └── REPOSITORY_STRUCTURE.md
└── registry/
    └── <maintainer-slug>/
        ├── maintainer.yaml
        ├── <artifact-id>/
        │   ├── artifact.yaml
        │   └── SKILL.md
        └── <pack-id>/
            ├── artifact.yaml
            └── skills/
                ├── <skill-id>/SKILL.md
                └── <skill-id>/SKILL.md
```

## Maintainer Namespace

Each maintainer owns exactly one directory under `registry/`.

Example:

```text
registry/golemcore/
```

Required file:

```text
registry/<maintainer-slug>/maintainer.yaml
```

Suggested fields:

```yaml
schema: v1
id: golemcore
display_name: Golemcore
github: golemcore
contact: team@golemcore.dev
```

Rules:

- `id` must match the directory name.
- The maintainer slug must be unique across the repository.
- Slugs must use lowercase letters, digits, and hyphens only.
- Display names may change; slugs should not.

## Standalone Skill Artifact

Directory layout:

```text
registry/<maintainer-slug>/<artifact-id>/
  artifact.yaml
  SKILL.md
```

Example:

```text
registry/golemcore/code-reviewer/
  artifact.yaml
  SKILL.md
```

Suggested `artifact.yaml`:

```yaml
schema: v1
type: skill
maintainer: golemcore
id: code-reviewer
version: 1.0.0
title: Code Reviewer
description: Review code for correctness, risks, and maintainability.
```

Rules:

- `type` must be `skill`.
- `maintainer` must match the parent namespace.
- `id` must match the artifact directory name.
- The artifact contains exactly one `SKILL.md` at the artifact root.

## Pack Artifact

Directory layout:

```text
registry/<maintainer-slug>/<artifact-id>/
  artifact.yaml
  skills/
    <skill-id>/SKILL.md
    <skill-id>/SKILL.md
```

Example:

```text
registry/golemcore/devops-pack/
  artifact.yaml
  skills/
    deploy-review/SKILL.md
    incident-triage/SKILL.md
```

Suggested `artifact.yaml`:

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

- `type` must be `pack`.
- `maintainer` must match the parent namespace.
- `id` must match the artifact directory name.
- Every skill declared in `artifact.yaml` must exist on disk.
- Every `skills/<skill-id>/SKILL.md` must have a unique `skill-id` within the pack.

## `SKILL.md`

`SKILL.md` remains the source of truth for an individual skill definition.

It should contain:

- YAML frontmatter for skill metadata
- the executable prompt or instructions body
- any usage notes required by the runtime

Recommended frontmatter fields:

```yaml
name: code-reviewer
description: Review code for correctness and maintainability.
model_tier: smart
```

Rules:

- The runtime-facing skill identity should stay stable once published.
- A standalone artifact should usually keep `name` aligned with the artifact id.
- A skill inside a pack should keep `name` aligned with its pack-local `skill-id` unless there is a strong migration reason not to.

## Naming Rules

Use the following slug pattern for:

- maintainer ids
- artifact ids
- pack-local skill ids

Pattern:

```text
[a-z0-9][a-z0-9-]*
```

Do not use:

- spaces
- underscores
- uppercase letters
- mutable display names as identifiers

## Identity Rules

Published identity should be immutable.

- Maintainer namespace should not be renamed casually.
- Artifact ids should not be reused for unrelated content.
- Pack-local skill ids should stay stable between versions.
- If a breaking identity change is unavoidable, create a new artifact id and document the migration.

## Versioning

Each artifact should declare a `version` in `artifact.yaml`.

Guidelines:

- bump the version for every published content change
- use semantic versioning when possible
- do not change published content without also changing the artifact version

Suggested interpretation:

- patch: fixes, clarifications, non-breaking metadata changes
- minor: new capabilities or new skills in a pack
- major: breaking prompt contract, removals, or incompatible identity changes

## Recommended Future Metadata

Tooling may grow over time. The following fields are recommended for future compatibility:

```yaml
compatibility:
  golemcore: ">=0.15.0 <0.16.0"
source:
  repository: https://github.com/golemcore/golemcore-skills
```

Optional pack-oriented metadata:

```yaml
tags:
  - devops
  - operations
```

## Review Guidelines

Reviewers should validate:

- maintainer namespace ownership
- artifact path correctness
- artifact type correctness
- version increments
- pack skill list consistency
- English-only documentation and metadata
- stable identifiers

## Anti-Patterns

Avoid the following:

- flat repositories where author ownership must be guessed
- packs without an explicit artifact manifest
- hidden skills not declared by pack metadata
- changing artifact contents without changing the version
- renaming published ids without a migration story

## Minimal Publishing Checklist

1. Add or update `registry/<maintainer-slug>/maintainer.yaml` if needed.
2. Add one artifact directory.
3. Add `artifact.yaml`.
4. Add one `SKILL.md` for a standalone artifact, or a `skills/` tree for a pack.
5. Confirm all identifiers use stable lowercase slugs.
6. Confirm all text is written in English.
7. Commit using Conventional Commits.
