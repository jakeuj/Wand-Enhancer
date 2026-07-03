# Wand Enhancer Repository Invariants

## Remote Web Panel

- Keep the default local remote port at `3223`.
- Keep shared web protocol version, port, and HTTP/WS paths in `web-panel/protocol/web-contract.json`.
- Keep bridge-only IPC channels, websocket opcodes, and renderer injection delays in `web-panel/bridge/src/constants.ts`.
- Do not duplicate the presentation URL or port in C#.
- Keep the embedded panel small; the WPF patcher embeds `web-panel/dist` and injects it into Wand `app.asar`.
- Do not ship mock data, debug routes, sourcemaps, local fonts, heavy icon libraries, or runtime class helper packages in production dist.
- Keep mock/demo data behind `import.meta.env.DEV` dynamic imports.
- Allow React-compatible source imports, but preserve Preact production aliases in `web-panel/vite.config.ts`.
- Use Tailwind CSS and lightweight primitives from `web-panel/src/shared/ui/`.

## Bridge And Renderer Scripts

- Author the Electron bridge in TypeScript under `web-panel/bridge/src/`.
- Bundle/minify production runtime into `web-panel/dist/bridge.cjs` with `pnpm run build:bridge`.
- Do not copy bridge source into Wand or embed it as ASAR resources.
- Keep default renderer script sources in `web-panel/bridge/scripts/default/`.
- Bundle/minify default renderer scripts into `web-panel/dist/renderer-scripts/` with `pnpm run build:bridge`.
- Copy selected custom user scripts from `PatchConfig.CustomScriptPaths`; accept only existing `.js` files.
- Continue supporting a local `renderer-scripts/` folder beside the patcher exe as an advanced fallback.

## Remote Popup Cleanup

- Keep remote tooltip links and every rendered `remote-qr-code` redirected by `web-panel/bridge/scripts/default/remote-popup-cleanup.js`.
- Reuse Wand's loaded QR renderer through the webpack runtime.
- Keep the local URL visible as a fallback.
- Hide the Pro onboarding remote mobile app card.
- Do not reintroduce C# ASAR patches for tooltip URL or QR component; a changed UI bundle must not fail the whole remote-panel patch.

## Installed Apps And Game Status

- `installed-apps-sync.js` resolves Wand renderer services/store and publishes `My Games` through `wand-remote-installed-apps`.
- Mirror Wand `my_games` source criteria: catalog games from `installedGameVersions`; extra installed unsupported titles from `correlatedUnavailableTitles` whose `games[].correlationIds` match `installedApps`.
- If live store `correlatedUnavailableTitles` is missing or empty, fall back to Wand `/v3/unavailable_titles` correlation lookup through the renderer API client.
- Do not degrade to raw install entries or an empty `My Games` list when correlation lookup is available.
- Prefer game artwork in `imageUrl`.
- Prefer Wand client icon CDN shape `https://api-cdn.wemod.com/steam_community/<steamAppId>/client_icon/96.webp` whenever matched metadata exposes a Steam AppID, regardless of install platform.
- Search nested `steam*` metadata before falling back to installed Steam `sku`; do not assume a flat `steamAppId`.
- If metadata has no usable icon, fall back to Wand sidebar DOM `.sidebar-game-row-image` background-image keyed by `titleId` parsed from `data-tooltip-trigger-for`.
- Keep web panel `GameCover` tolerant of broken artwork URLs and able to fall back to text cover.
- Forward lifecycle state through `wand-remote-game-status`: `game-launched` / `game-ended` from launch monitor service, trainer runtime from running-trainer visibility service.
- When Wand does not emit `game-launched` but trainer visibility already indicates active trainer state, synthesize a running session.
- Websocket `hello` must still send cached `installed_apps` and `game_status` even when no trainer snapshot is active; do not return early after `trainer_changed`.

## Remote Play And Stop

- Remote Play/Stop uses websocket `remote_command`.
- Bridge forwards commands over `wand-remote-command` / `wand-remote-command-response`.
- `installed-apps-sync.js` resolves Wand trainer API plus trainer service to launch a trainer for `gameId` or end the current trainer.
- Remote Play must construct Wand's real trainer launch request class, currently observed as `69482.vO`, before calling `trainerService.launch(...)`.
- Do not pass a plain object to `trainerService.launch`; it can launch the process while breaking `getMetadata(vO)`-based trainer state.

## Pro Activation Patches

- Treat Pro activation as an independent C# ASAR patch: `EPatchType.ActivatePro`.
- Preserve patches that inject `subscription:{period:"yearly",state:"active"}` before account responses reach the store.
- Account-returning service methods include `getUserAccount`, `setAccountWandBrandExperience`, and `setAccountLanguage`.
- Preserve the `setAccountReducer` patch so `ACTION_SET_ACCOUNT` writes keep Pro even when bypassing account service methods.
- Remember Wand Pro check is `am(account) = !!account.subscription`; flags such as `512` are irrelevant.
- If Wand changes method bodies, re-derive regexes against the live `app-*.bundle.js`.
- Do not trust `.source/new`; it can be a different version.

## ASAR Patch Pipeline

- Preserve and restore backups for both `resources/app.asar` and `resources/app.asar.unpacked`.
- Inject `web-panel/dist` as `remote-panel/`; it must already contain `bridge.cjs` and generated default renderer scripts under `renderer-scripts/`.
- Copy selected/local custom renderer scripts under `remote-panel/renderer-scripts`.
- Do not commit extracted `.source/` or `.sources/`; recreate extraction only for reverse-engineering sessions.
- `AsarSharp.AsarExtractor.ExtractAll` must skip unpacked entries when source path equals destination.
- `ExtractAll` must silently skip unpacked entries whose source is missing on disk.
- Do not reintroduce hard failures for in-place self-copy or missing unpacked entries.
- `DevToolsOnF12` must anchor on the Electron main-process `<app>.whenReady().then(` site and attach a `before-input-event` hook to every `BrowserWindow.webContents`.
- Do not patch the renderer keydown listener for DevTools; the minified `ACTION_OPEN_DEV_TOOLS` dispatch site is unstable.

## Web Panel State And UI

- Store pinned cheats per game with `pinned-storage.ts` under `wand-remote.pinned-cheats.v1:<gameId>`.
- Render pinned cheats as a virtual `pinned` category at the top while preserving normal category placement.
- Store custom quick presets per trainer/game with `preset-storage.ts` under `wand-remote.presets.v1:<gameId-or-trainerId>`.
- Save persistent cheat values only; never include one-shot `button` cheats in presets.
- Route all `localStorage` access in `web-panel/src/` through `web-panel/src/shared/storage.ts`.
- Use `loadJson`, `saveJson`, `loadStringSet`, and `saveStringSet`; do not reintroduce local `try/catch` plus `JSON.parse` duplication.
- Derive trainer/game storage IDs through `getTrainerStorageId(trainer)`.
- Do not inline the `gameId -> titleId -> trainerId -> global` precedence.
- Follow the `E*` enum convention for UI string-union types. Keep wire string values on the right-hand side of enum members.
- Keep reducer action tags as discriminated-union string literals.
- Keep cheat input controls one-per-file under `web-panel/src/trainer/controls/`.
- Keep shared `SliderTrack`, `StepButton`, and `ControlInternalProps` in `controls/shared.tsx`.
- Keep number formatting helpers in `controls/format-number.ts`.
- Keep `controls/CheatControl.tsx` as a thin dispatcher keyed by `ECheatType`.
- For mobile drawer performance, keep drawer panels and nested glass controls blur-free under coarse pointers.
- Do not add per-row `backdrop-blur-*` inside drawer lists.

## Validation Commands

- Web panel build: `cd web-panel && pnpm run build`.
- Bridge syntax: `node --check web-panel/dist/bridge.cjs`.
- Renderer script syntax: `node --check web-panel/dist/renderer-scripts/remote-popup-cleanup.js`.
- Production dist forbidden-string scan: `rg -n "mock-instance|Mock Adventure|Simulation|Debug session|mock=1|demo-session|vite\.svg|tailwind-merge|class-variance-authority|clsx" web-panel/dist`.
- Full executable build: `.\build.cmd` from repo root.
