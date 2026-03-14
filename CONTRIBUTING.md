# Contributing

This repository stores reusable Golemcore skills and packs. Contributions must keep the registry predictable for both humans and automation.

## Language

- Use English for all documentation, metadata, and pull request descriptions.

## What Can Be Added

- maintainer namespaces
- standalone skill artifacts
- pack artifacts containing multiple skills
- repository-level documentation

## Branch Naming

Use descriptive topic branches.

Examples:

- `feat/add-code-reviewer`
- `feat/add-devops-pack`
- `docs/repository-structure`
- `fix/update-artifact-metadata`

## Commit Messages

Conventional Commits are mandatory.

Recommended scopes:

- `feat(skill): ...`
- `feat(pack): ...`
- `feat(maintainer): ...`
- `docs: ...`
- `fix(skill): ...`
- `fix(pack): ...`
- `chore: ...`

Examples:

- `feat(skill): add golemcore/code-reviewer`
- `feat(pack): add golemcore/devops-pack`
- `feat(maintainer): add golemcore namespace metadata`
- `docs: describe repository structure`

## Pull Request Expectations

Each pull request should explain:

- what was added or changed
- which maintainer namespace is affected
- whether the artifact is a standalone skill or a pack
- whether existing artifact references are changed
- whether consumers need to update any install metadata

## Repository Rules

- Use stable lowercase slugs for maintainers, artifacts, and skill directories.
- Do not move or rename published artifacts without an explicit migration plan.
- Do not reuse an existing artifact identifier for different content.
- Keep one artifact per directory.
- Keep each `SKILL.md` self-contained and reviewable.

## Before Opening a Pull Request

1. Confirm that the path matches the repository structure in [docs/REPOSITORY_STRUCTURE.md](docs/REPOSITORY_STRUCTURE.md).
2. Confirm that metadata and examples are written in English.
3. Confirm that the commit history uses Conventional Commits.
4. Confirm that the pull request title also follows Conventional Commits when possible.
