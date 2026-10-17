# future-projects

Tracking and planning workspace for the **concept → planned → active** development pipeline
(used by the [`CoderManSkills`](https://github.com/CoderMan73/CoderManSkills) `plan-to-active`
skill). Planning documents live here; generated code repositories live elsewhere under
`E:\Coding_Projects\<name>\`.

## Layout

```
public/            # committed planning documents (reviewed / shareable)
  concept/         # CONCEPT.md drafts (rough ideas, under discovery)
  planned/         # RESEARCH.md + PLAN.md (planning complete, awaiting approval)
  active/          # projects currently in development (CONCEPT/RESEARCH/PLAN present)
  archived/        # completed or abandoned projects
private/           # gitignored — never committed (proprietary assets, homelab notes,
                   # draft material, post-mortems)
```

`.gitignore` ignores the entire `private/` tree, so anything local-only stays local.

## Stages

| Stage | Folder | Contents | Status |
|---|---|---|---|
| Concept | `public/concept/<name>/` | `CONCEPT.md` | rough, unrefined |
| Plan | `public/planned/<name>/` | `CONCEPT.md`, `RESEARCH.md`, `PLAN.md`, `DISCOVERY.md` | plan approved; awaiting scaffold |
| Active | `public/active/<name>/` | relocated planning docs + link to code repo | in development |
| Archived | `public/archived/<name>/` | moved here on completion/abandonment | done |

## Lifecycle

1. Drop a `CONCEPT.md` in `public/concept/<name>/` (created by the `concept-architect` skill).
2. `plan-to-active` relocates it to `public/planned/<name>/` and scaffolds a code repo.
3. On acceptance, it moves to `public/active/<name>/` (with a `MANIFEST.md` linking to the
   code repo) and development begins in the code repo.
4. On completion, relocate to `public/archived/<name>/`.

## Current projects

**Concept:** `easy-farms`, `fast-biomes`, `mkw-race-tracker`, `modpack-seedfinder`,
`nuzlocke-bot`, `personal-website`, `playlistdle`, `pokebottrainer`, `procedural-tools`,
`team-perfection-tool`, `terraria-seedfinder`.

**Active:** `craftlatro`, `craftleste`, `craftraria`, `CoderManSkills`, `future-projects`.

## Notes for agents

- Do not commit to `private/`. It is gitignored and may contain proprietary game assets
  (ROMs, SDKs) and binaries that must not leave the local machine.
- Planning docs (`CONCEPT.md`, `RESEARCH.md`, `PLAN.md`) are the source of truth here;
  never duplicate them into a code repo — reference them by tracking path (recorded in the
  generated `AGENTS.md`).
- Post-mortems live in `private/post-mortems/` (local only); do not relocate them into `public/`.

## License

GNU General Public License v3.0 — see `LICENSE`.
