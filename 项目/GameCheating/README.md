---
type: project
tags: [project, renpy, save-editor]
date: 2026-08-19
project: GameCheating
status: active
---

# GameCheating

GameCheating is a local-first collection of related Ren'Py save-editing tools. The repository groups the generic CLI core with the Agent17 and Eternum-specific GUI tools because they share archive parsing, pickle safety, integer patching, and Ren'Py signing concerns.

## Repository

- Repository: `D:\Desktop\Craft\GameCheating`
- Project reference: `D:\Desktop\Craft\GameCheating\PROJECT_REFERENCE.md`
- Version-control link contract: `D:\Desktop\Craft\GameCheating\KNOWLEDGE_BASE.md`

## Stable facts

- The generic core handles Ren'Py protocol 2 without frames and protocol 5 with `FRAME`.
- The Agent17 tool targets the existing Ren'Py 8.5 save format.
- The Eternum tool targets Eternum 0.9.0 / Ren'Py 8.3.2 protocol 2 saves.
- Original saves are never overwritten; uncertain or unsupported data fails closed or remains read-only.
- Module-specific behavior and open work stay in the repository documents, especially each module's `PROJECT_REFERENCE.md`, `CONSENSUS.md`, `TO-TICKETS.md`, and `DEV_LOG.md`.

## Related notes

- [[persona]]
- [[项目/README|项目注册表]]
