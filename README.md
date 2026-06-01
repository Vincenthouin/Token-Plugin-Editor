# Token Plugin & Editor

Outils de synchronisation des design tokens Somfy entre Figma et GitHub.

Les **tokens** (source de vérité) vivent dans un repo séparé :
👉 https://github.com/Vincenthouin/tokens-poc

Ce repo contient les deux outils qui les éditent.

```
Token-Plugin-Editor/
├── figma-plugin/   ← plugin Figma "Somfy Token Sync"  (bidirectionnel)
└── editor/         ← app desktop (Electron + React)
```

## 🎨 `figma-plugin/`

Plugin Figma bidirectionnel entre le JSON GitHub (`tokens-poc`) et Figma.

**JSON → Figma (pull)**
- Variables Figma (couleurs Light/Dark, dimensions, font weights/sizes/family)
- Text Styles (fontSize lié aux Variables)
- Effect Styles (shadows)
- Détection des diffs (Added / Modified / Removed), résolution d'aliases, nommage auto.

**Figma → JSON (push)**
- Détection des dérives locales : variables/styles modifiés, renommés, supprimés ou ajoutés directement dans Figma.
- Création de PR GitHub en un clic (5 appels REST natifs, pas de backend).
- Auto-fetch au boot + auto-refresh post-merge via polling SHA.
- Bandeau "PR pending merge" persistant qui survit aux reloads et bloque Check/Apply tant que la PR n'est pas mergée.

→ Détails : [`figma-plugin/README.md`](./figma-plugin/README.md)

## 💻 `editor/`

App desktop (Electron + React) pour éditer les tokens via UI :
- Couleurs avec color picker dual Light/Dark
- Filtrage par catégorie (Color / Spacing / Radius / Shadow / Font)
- Renommage avec mise à jour auto des références
- Picker d'aliases avec autocomplete
- Multi-projets (GitHub / local), undo/redo, suppression en bulk, tolérance valeurs vides
- Création automatique de PR à chaque sauvegarde, historique + rollback via PR de revert

→ Détails : [`editor/README.md`](./editor/README.md)

## Workflow

```
┌─────────────┐       ┌────────────────┐       ┌──────────────┐
│  editor/    │ ─PR─▶ │   tokens-poc   │ ◀PR/API▶ figma-plugin │
│  (UI desk)  │       │ tokens/*.json  │        │  (sync Figma)│
└─────────────┘       └────────────────┘       └──────────────┘
```

Les deux outils ciblent la branche `develop` de `tokens-poc` ; promotion vers `main` = action manuelle.

## Setup

### Plugin Figma
```bash
cd figma-plugin
npm install
npm run build        # ou: npm run watch
```
Puis dans Figma : Plugins → Development → Import plugin from manifest → choisir `figma-plugin/manifest.json`.

### Éditeur desktop
```bash
cd editor
npm install
npm run dev          # mode dev
npm run build:mac    # produit le DMG dans editor/release/
```

## Sécurité

- Ne **jamais** committer un PAT GitHub dans le code ou la config.
- Les PAT sont stockés en local : `clientStorage` (Figma) ou `electron-store` (editor).
- Phase 3 prévue : GitHub App + Cloudflare Worker pour ne plus exposer de PAT côté client.
