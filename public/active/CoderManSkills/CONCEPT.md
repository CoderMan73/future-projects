# CONCEPT.md

## Project Name

CoderManSkills

## Objective

A personal library of agent skills that improve AI behavior through structured SKILL.md files. Each skill lives in its own directory under `SKILLS/` and provides behavioral guidelines, conventions, and workflows to make the agent less generic and more aligned with the user's standards. This concept document records the existing repo for housekeeping purposes in the future-projects organization.

## Technical Goals

- Maintain a collection of self-contained agent skills, each defined by a `SKILL.md` file with YAML frontmatter and behavioral guidelines.
- Enforce consistent conventions across all skills: frontmatter format, tone, structure, and naming.
- Support the full skill lifecycle: creation (`create-skill`), usage (`concept-architect` and others), and iterative improvement (`improve-skill`).

## Scope

### Existing Skills

1. **concept-architect** (`SKILLS/concept-architect/SKILL.md`) - Refines vague project ideas into concrete, buildable technical specifications through iterative clarification and documentation. Produces a `CONCEPT.md` that records the user's objective, technical goals, restrictions, and preferences. Stops at documentation; does not execute the concept.

2. **create-skill** (`SKILLS/create-skill/SKILL.md`) - Generates a new `SKILL.md` on demand in the established project style. Takes a skill name and description, optionally a scope note, and writes the file into `SKILLS/<name>/`. Enforces frontmatter, body structure, formatting, and tone conventions.

3. **improve-skill** (`SKILLS/improve-skill/SKILL.md`) - Performs a post-mortem analysis of a skill interaction (via session title lookup with `kilo_local_recall` or a raw transcript) to drive continuous improvement. Produces optimized system instructions, refined behavioral constraints, and a strategic framework.

### Workflow

Skills are invoked from within an agent session by referencing the skill name (e.g., `@concept-architect`). The `create-skill` skill writes new skills into the `SKILLS/` directory. After creation, the agent session is reloaded to pick up the new skill. The `improve-skill` skill reviews past interactions and produces refinements that can be applied back to a skill's `SKILL.md`.

## Design Decisions

- **File format**: Each skill is a single `SKILL.md` file in its own directory, using YAML frontmatter for metadata (name, description) followed by a markdown body.
- **Naming**: Directory names use lowercase letters, numbers, and hyphens only, matching the `name` field in frontmatter.
- **Scope**: Each skill is focused and single-purpose. Complex multi-stage workflows are decomposed into separate skills.
- **Convention enforcement**: The `create-skill` skill encodes and enforces the established conventions (frontmatter, body structure, formatting, tone) so that every generated skill matches the project style.
- **Self-referential**: The repo contains skills that can modify the repo itself (e.g., `create-skill` writes new skills, `improve-skill` reviews existing ones).

## Constraints

- Skill names: lowercase letters, numbers, hyphens only; no leading or trailing hyphen; at most 64 characters.
- Descriptions: one sentence, at most 1024 characters.
- New skills require reloading the agent session to be loaded; no config change is needed since the skills path already covers the `SKILLS/` parent directory.
- Skills are loaded from the `SKILLS/` directory.
- No auto-commit or auto-deploy of generated skills; the user must explicitly commit changes.

## Unresolved Design Decisions

- Whether to support multi-file skills (beyond a single `SKILL.md`) for complex skills needing reference materials or scripts.
- Whether to implement a validation/linting step to verify generated `SKILL.md` files conform to all conventions before writing.
- Whether the `.kilo/worktrees/` directory (containing a `clean-gardenia` worktree) represents an active development branch or a deployment copy.

## User Preferences

- Skills are written in markdown, not code. No compilation or build step required beyond reloading the session.
- The repo uses a worktree workflow (`.kilo/worktrees/clean-gardenia`) suggesting parallel branch development for skill iterations.
- MIT license (shared LICENSE file).

## Repository Structure

```
CoderManSkills/
  .kilo/
    worktrees/
      clean-gardenia/
        SKILLS/
          concept-architect/SKILL.md
          create-skill/SKILL.md
          improve-skill/SKILL.md
        LICENSE
  SKILLS/
    concept-architect/SKILL.md
    create-skill/SKILL.md
    improve-skill/SKILL.md
  LICENSE
```

---

This document is a draft proposal. It has not been validated against the user's intent and should be reviewed and confirmed before being treated as finalized.
