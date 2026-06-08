# Docs Notes — Index

_This directory contains the team's working notes, annotations, and summaries of the official Stake Engine documentation._

---

## Source Docs

| Source | URL |
|--------|-----|
| Official docs (browser only) | https://stake-engine.com/docs |
| Markdown mirror (64 files, searchable) | https://github.com/jovanpanetie/stake-engine-md-doc |

> **Done — already cloned.** The mirror is at `docs/stake-engine-md-doc/` — 64 searchable markdown files. Snapshot from April 2026.
> To update: `cd docs/stake-engine-md-doc && git pull`

---

## Notes Files in This Directory

### Plain-English Overviews (start here)
| File | Covers | Primary Reader |
|------|--------|----------------|
| [how-games-work-sdk-stack.md](how-games-work-sdk-stack.md) | How slot games work end-to-end using the Stake Engine SDK | Everyone |
| [how-games-work-hybrid-stack.md](how-games-work-hybrid-stack.md) | How non-slot games work end-to-end using our Phaser hybrid stack | Everyone |

### Technical Reference Notes
| File | Covers | Primary Reader |
|------|--------|----------------|
| [rgs-notes.md](rgs-notes.md) | API flow, wallet endpoints, session handling, money/currency, error codes | Zacke |
| [math-sdk-notes.md](math-sdk-notes.md) | Python math framework, game structure, simulation, file format | Zacke |
| [frontend-sdk-notes.md](frontend-sdk-notes.md) | PixiJS/Svelte SDK, events, file structure, Storybook | Tom |
| [approval-notes.md](approval-notes.md) | Submission checklist, quality rankings, replay requirements, tile assets | Tiger + Ollie |

---

## How to Use These Files

- Add questions, confusions, and gotchas as you read — don't wait until you fully understand something
- Mark things **[CONFIRMED]** once you've tested them in practice
- Mark things **[UNCLEAR]** if the docs are ambiguous — Tiger can follow up with Stake Engine
- Keep notes short and practical — what Zacke needs to build, what Ollie needs to review
