# Viora

**Your Media. Your Way.**

Viora is a personal video resource aggregation platform. It brings lawful live TV, on-demand video, educational programs, documentaries and personal media into one customizable catalog with a Web-first interface suitable for TV browsers.

## Product goals

- Browse by **content** and by **source** using configurable up-to-three-level category trees.
- Search across multiple providers in one place; unify the same work while retaining alternative playback sources.
- Prefer **direct in-app playback** when technically supported and authorized; otherwise use official embedded players or external links.
- Provide poster-wall discovery, watch progress, favorites and D-pad-friendly TV navigation.
- Keep sources and adapters replaceable without losing the user's taxonomy or library.

## Planned technology

TypeScript, React, Vite, Tailwind CSS, NestJS, PostgreSQL, Prisma, pnpm workspaces, Docker Compose. Modular monolith backend. Web first, Android TV native app only if later justified.

## Start here

- [Product requirements and architecture](docs/VIORA_PROJECT_BLUEPRINT.md)
- [Phase 0 Codex implementation brief](docs/CODEX_PHASE0.md)

**Status:** planning / initial scaffold. There is no runnable application in this repository yet.

## Legal and technical boundaries

Use authorized streams, public media APIs and personal libraries. Respect DRM, access controls, platform terms, CORS and embedding rules. Never assume every indexed video can be played inside Viora.
