---
name: wand-enhancer-repo
description: "Repository guide for Wand Enhancer, a .NET Framework WPF patcher with an embedded Vite/Preact remote web panel. Use when Codex works inside H:\\repos\\Wand-Enhancer, especially for web-panel, bridge, renderer scripts, ASAR patching, Pro activation patches, build output, Rider/MSBuild/CMake native helper issues, localStorage, fork branch strategy, GitHub Actions executable builds, or validation tasks."
---

# Wand Enhancer Repo

Use this skill to make narrow, repo-compatible changes to Wand Enhancer without breaking the patch pipeline.

## Working Flow

1. Read the touched files first and preserve user changes in the working tree.
2. Keep bridge, frontend, protocol, and C# constants aligned through the existing shared sources.
3. Avoid broad refactors; this repo patches a minified Electron app, so stable anchors and fallback behavior matter.
4. Read `references/repo-invariants.md` before editing patch-sensitive code in `web-panel/`, `WandEnhancer/`, `AsarSharp/`, or the native helper.
5. Read `references/fork-actions-workflow.md` before changing remotes, branch strategy, fork setup, GitHub Actions workflow dispatch, or executable artifact handling.
6. If `docs/` or `.claude/rules/frontend-conventions.md` exists in the checkout, read relevant files before relying on their rules.

## Build And Validation

For web panel work, run from `web-panel/`:

```powershell
pnpm run build
node --check dist\bridge.cjs
node --check dist\renderer-scripts\remote-popup-cleanup.js
rg -n "mock-instance|Mock Adventure|Simulation|Debug session|mock=1|demo-session|vite\.svg|tailwind-merge|class-variance-authority|clsx" dist
```

Treat the final `rg` exit code `1` as success when it means no matches.

For a full Release executable, run from the repo root:

```powershell
.\build.cmd
```

If `cmake` is installed with Visual Studio Build Tools but missing from `PATH`, temporarily prepend the Visual Studio CMake `bin` directory, then rerun the same build command.

The Release output is `WandEnhancer\bin\Release\WandEnhancer.exe`.

For Rider Debug runs, remember that `WandEnhancer.csproj` validates the native proxy DLL before compiling. Debug builds need `.tmp\cmake\asar-fuses-bypass\Debug\version.dll`; Release builds need the Release equivalent. If Rider reports `Proxy DLL not found: ..\.tmp\cmake\asar-fuses-bypass\Debug\version.dll`, configure/build the native helper for Debug before running the .NET project:

```powershell
cmake -S tools\asar-fuses-bypass -B .tmp\cmake\asar-fuses-bypass -A x64
cmake --build .tmp\cmake\asar-fuses-bypass --config Debug
```

When `cmake` is not available in the shell, use the Visual Studio bundled CMake under `<VS install>\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin`, or add that directory to the user `PATH` and restart Rider.

If Release build fails while copying `WandEnhancer.exe`, check whether the app is already running from `WandEnhancer\bin\Release`; a running instance locks the output file.

Never commit or push generated executables or build folders (`.exe`, `bin/`, `obj/`, `dist/`). Treat GitHub Actions artifacts as the downloadable executable source.
