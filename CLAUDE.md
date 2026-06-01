# Token Plugin & Editor — Project memory

This file is auto-loaded by Claude Code at session start.
Keep it terse, source-of-truth, no historical narrative.

## What this repo is

Outils d'édition des design tokens Somfy. Les tokens eux-mêmes vivent
dans un repo séparé (source de vérité).

```
Token-Plugin-Editor/
├── figma-plugin/   ← plugin Figma "Somfy Token Sync" (bidirectionnel)
└── editor/         ← app desktop (Electron + React)
```

GitHub: https://github.com/Vincenthouin/Token-Plugin-Editor

Tokens (JSON W3C, source of truth) : https://github.com/Vincenthouin/tokens-poc
- Branche cible des PR auto = `develop` de `tokens-poc`.
- Les outils lisent `tokens/somfy-tokens.json` sur cette branche.

## Branches & sync policy

- `main` = version stable des deux outils.
- `develop` = branche par défaut des configs plugin/éditeur (PRs ouvertes ici).
- **Don't push to origin/main without explicit user instruction.**

## Conventions

- **Language** : code en anglais (commits, identifiants, JSON) ;
  commentaires et UI parfois en français (l'utilisateur est français).
- **Commits** : titre impératif, ~70 chars max. Toujours terminer par
  `Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>`.
- Pas de PAT en clair, jamais. Storage = `clientStorage` (Figma) /
  `electron-store` (editor).

## `figma-plugin/`

- Main thread : `code.ts` (~2000 lignes).
- UI : `ui.html` (vanilla JS, ~2000 lignes).
- Manifest : `manifest.json`.
- Build : `npm run build` → `code.js` (loadé par Figma).

Points sensibles :
- Aliases résolus récursivement via path lookup dans le JSON.
- Couleurs Light/Dark : `$value = { light, dark }` (jamais `$extensions.modes`).
- Diff Figma → JSON détecte add/modify/rename/delete via SHA + name map.
- PR pending-merge survit aux reloads (state persisté `clientStorage`).

## `editor/`

- Electron + React + TypeScript + Vite.
- IPC : `src/main/index.ts` (handlers Octokit + projet CRUD).
- Renderer : `src/renderer/` (UI table, sidebar tree, ValueEditor, etc.).
- Storage local : `src/main/projectStore.ts` (electron-store).
- Build DMG (arm64) : `npm run build:mac` → `editor/release/`.

Points sensibles :
- Multi-projets : GitHub OR local JSON.
- Undo/redo via `useTreeHistory`.
- Rename d'un token = mise à jour auto des aliases qui le référencent.
- Tolère `$value: ""` (token vide) sans planter.

## W3C Design Tokens shape (rappel)

- Group = `{ "child": ... }` (pas de `$value`).
- Token = `{ "$type": "color", "$value": "...", "$description"?, "$extensions"? }`.
- Alias = `"{primitives.loop.color.background.main}"`.
- Layers historiques : `primitives` / `semantic` / `composite` / `component`,
  mais pas hardcodés — toute clé top-level non-`$` est traitée comme une layer.
