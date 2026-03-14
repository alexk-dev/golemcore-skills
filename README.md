# golemcore-skills

Canonical repository for Golemcore skills and skill packs.

This repository defines the structure, naming rules, and contribution workflow for publishing reusable Golemcore skills. It is designed to support:

- standalone skills maintained by individual authors
- packs that ship multiple related skills together
- multiple maintainers with stable namespaces
- future registry tooling that can install either a single skill artifact or a pack artifact

## Status

The repository is currently documentation-first. The layout described here is the canonical contract for future content. Consumer tooling may adopt the full contract incrementally.

## Repository Goals

- provide one predictable home for community and first-party skills
- avoid naming conflicts across maintainers
- support both single-skill artifacts and multi-skill packs
- make artifact ownership, versioning, and provenance explicit
- keep repository review and automation simple

## Canonical Layout

```text
registry/
  <maintainer-slug>/
    maintainer.yaml
    <artifact-id>/
      artifact.yaml
      SKILL.md
    <pack-id>/
      artifact.yaml
      skills/
        <skill-id>/SKILL.md
        <skill-id>/SKILL.md

docs/
  REPOSITORY_STRUCTURE.md
```

## Artifact Types

### Standalone skill

A standalone skill is one installable artifact containing exactly one `SKILL.md`.

Example:

```text
registry/golemcore/code-reviewer/
  artifact.yaml
  SKILL.md
```

Canonical artifact reference:

```text
golemcore/code-reviewer
```

### Pack

A pack is one installable artifact that contains multiple skills.

Example:

```text
registry/golemcore/devops-pack/
  artifact.yaml
  skills/
    deploy-review/SKILL.md
    incident-triage/SKILL.md
```

Canonical artifact reference:

```text
golemcore/devops-pack
```

Canonical skill reference inside a pack:

```text
golemcore/devops-pack/deploy-review
```

## Required Documents

- [Repository Structure](docs/REPOSITORY_STRUCTURE.md): full layout and metadata contract
- [Contributing Guide](CONTRIBUTING.md): contribution rules, commit format, and PR expectations

## Contribution Rules

- All documentation and metadata must be written in English.
- Each artifact must live under exactly one maintainer namespace.
- File and directory names must use stable lowercase slugs.
- Repository changes must use Conventional Commits.
- Pull requests to `main` are required; direct commits to `main` are not allowed.
- Pull requests should clearly state whether they add a maintainer, a standalone skill, a pack, or repository-level documentation.

## Conventional Commits

Conventional Commits are required for this repository.

Examples:

- `feat(skill): add golemcore/code-reviewer standalone artifact`
- `feat(pack): add golemcore/devops-pack artifact`
- `docs: document repository structure and contribution workflow`
- `fix(skill): correct metadata for golemcore/code-reviewer`

## Recommended Workflow

1. Create a topic branch such as `feat/add-code-reviewer` or `docs/repository-structure`.
2. Add or update files under `registry/` and `docs/`.
3. Verify that names, paths, and metadata follow the repository contract.
4. Commit using a Conventional Commit message.
5. Open a pull request with a concise summary of the maintainer, artifact, and expected install surface.
