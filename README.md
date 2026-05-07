# sbux-mp-uniapp-development-skill

AI development skill pack for the Starbucks Mini-Program UniApp project, providing complete development standards and workflows for AI Agents.

## Design Principles

**Three-layer progressive disclosure**, loaded on demand to avoid injecting too much context at once:

```
SKILL.md (always loaded) → commands/ (loaded by scenario) → reference/ (loaded when diving deeper)
```

- **SKILL.md** — Project overview + naming conventions + directory structure + core spec summaries, auto-loaded by the Agent each conversation
- **commands/** — Step-by-step workflows for development scenarios, loaded on demand when users trigger commands
- **reference/** — Complete specs and code examples for each module, loaded when workflows need in-depth details

## Directory Structure

```
sbux-mp-uniapp-development-skill/
├── SKILL.md                    # Main file (always loaded)
├── examples.md                 # Complete code examples
├── commands/                   # Development command workflows
│   ├── page.md                 # /page — Create new page
│   ├── component.md            # /component — Create new component
│   ├── composable.md           # /composable — Write Composable
│   ├── service.md              # /service — Create new service domain
│   ├── store.md                # /store — Define/use Store
│   ├── deploy.md               # /deploy — Build & deploy
│   ├── monitor.md              # /monitor — Monitoring & reporting
│   └── use-sbux-mp.md          # /use-sbux-mp — End-to-end feature development
└── reference/                  # Detailed reference docs
    ├── composables.md           # Composable full architecture
    ├── component.md             # Component full patterns
    ├── services.md              # Service layer + multi-channel routing
    ├── store.md                 # Store full usage
    ├── transaction-store.md     # Transaction Store aggregate layer
    ├── shared-cache.md          # SharedCache cross-page cache
    ├── dialog.md                # Dialog Composables
    ├── static-assets.md         # Static asset management
    ├── deployment.md            # CI/CD pipeline
    └── monitor.md               # Tingyun SDK monitoring
```

## Command Quick Reference

| Command | Scenario |
|---------|----------|
| `/use-sbux-mp` | End-to-end business feature development (analyze → bottom-up creation → route registration → self-check) |
| `/page` | Create new page, Vue patterns, lifecycle management |
| `/component` | Create new component, dialog components, complex component splitting |
| `/composable` | Write main/sub composable, watch management |
| `/service` | Create new service domain, API integration, multi-channel requests |
| `/store` | defineStore definition/usage |
| `/deploy` | Build & deploy, CI/CD, environment variables |
| `/monitor` | Monitoring & reporting, Tingyun SDK, exception capture |

## Covered Standards

| Domain | Key Points |
|--------|-----------|
| Naming | File/directory camelCase, Hook `use` + PascalCase, platform-specific `name.platform.js` |
| Page | Main composable manages lifecycle, sub-composable pure logic; use Options `setup()` with `#ifdef` |
| Composable | Main/sub layering; watch must be cleaned up in `onUnmounted`; no lifecycle in sub-composables |
| Store | defineStore syntax; Mutation constants `UPDATE_XXX`; JS reads need `.value`; direct assignment prohibited |
| Service | Only requests + simple transforms; no routing/complex logic; supports multi-channel conditional compilation |
| Routing | `MOD`/`MOP`/`EC`/`USER` constants + `navigateTo()`; no hardcoded paths |
| Tracking | Encapsulate as functions in `track/`; naming `track[Element]Click/Expose` |
| Cache | SharedCache `defineCache` + `useCacheActions`; cross-page data sharing |
| Dialog | `useAlertDialog` / `useSlideDialog` / `usePromptDialog` |
| Static Assets | `imgUrl()` method; PNG/WebP adaptation; no inline base64 |
