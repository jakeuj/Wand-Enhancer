# Fork, Branch, And GitHub Actions Workflow

Use this reference for keeping a personal fork useful while preserving a clean upstream-tracking `master`.

## Remote Strategy

- Keep `master` clean and aligned with the original repository.
- Use `origin` for the user's fork: `https://github.com/jakeuj/Wand-Enhancer.git`.
- Use `upstream` for the original repository: `https://github.com/k1tbyte/Wand-Enhancer.git`.
- Disable upstream pushes with `git remote set-url --push upstream DISABLED`.
- When syncing original-author updates, use exactly this flow:

```powershell
git switch master
git fetch upstream
git merge --ff-only upstream/master
git push origin master
```

- If `merge --ff-only` fails, stop and inspect divergence instead of creating an automatic merge commit on `master`.
- After syncing `master`, create or rebase personal `codex/*` work branches as needed; do not use `master` for local build setup commits.

## Personal Work Branches

- Put local-only repo work on `codex/*` branches, not `master`.
- For local build setup and Codex skill work, use `codex/local-build-setup`.
- Keep commits intentionally split:
  - Codex skill updates, e.g. `chore: add Wand Enhancer Codex skill`.
  - Build environment changes, e.g. `build: pin local build prerequisites`.
- Stage explicit file paths; do not use broad staging when build outputs may exist.
- Do not commit `.exe`, `bin/`, `obj/`, `.tmp/`, extracted `.source/`, or generated `web-panel/dist`.

## Build Artifact Policy

- Do not treat `.exe`, `bin/`, `obj/`, `dist/`, or downloaded workflow outputs as source changes.
- Do not push build products as repo results.
- The executable users should download comes from the GitHub Actions artifact, not from committed files.
- Keep downloaded workflow artifacts under ignored local paths such as `.tmp/github-actions/run-<run-id>/`.

## Current Build Setup Decisions

- Target .NET Framework `v4.8.1` in both `AsarSharp/AsarSharp.csproj` and `WandEnhancer/WandEnhancer.csproj`.
- Keep `web-panel/pnpm-workspace.yaml` with:

```yaml
allowBuilds:
  esbuild: true
```

- This allows pnpm 11 installs to run the required `esbuild` postinstall scripts repeatably.

## GitHub Actions Executable Build

- Workflow file: `.github/workflows/build.yml`.
- Workflow name: `Build executable`.
- Trigger: `workflow_dispatch`.
- Branch to run for local setup builds: `codex/local-build-setup`.
- Artifact name: `WandEnhancer-unsigned`.
- Artifact contents: `WandEnhancer.exe`.

Prefer GitHub CLI when available:

```powershell
gh workflow run build.yml --repo jakeuj/Wand-Enhancer --ref codex/local-build-setup
gh run view --repo jakeuj/Wand-Enhancer --web
```

On this Windows host, `gh.exe` may be installed but not on `PATH`; check `C:\Program Files\GitHub CLI\gh.exe`.

Download an artifact from a successful run:

```powershell
gh run download <run-id> --repo jakeuj/Wand-Enhancer --name WandEnhancer-unsigned --dir .tmp\github-actions\run-<run-id>
```

## Workflow Registration Quirk

After a fresh fork, GitHub's Actions API may initially report zero workflows even when `.github/workflows/*.yml` files exist. Check the web Actions page. Once GitHub registers the workflows, `gh workflow list --all --repo jakeuj/Wand-Enhancer` should show `Build executable`.

If `gh workflow run build.yml` returns "workflow not found on the default branch":

1. Confirm `.github/workflows/build.yml` exists on `master`.
2. Open `https://github.com/jakeuj/Wand-Enhancer/actions`.
3. Enable workflows if GitHub shows a fork safety prompt.
4. Retry `gh workflow list --all --repo jakeuj/Wand-Enhancer`.
5. Trigger `build.yml` on the desired branch once the workflow appears.

## Local Build Troubleshooting

- If `.\build.cmd` cannot find `cmake`, look for Visual Studio's bundled CMake at `C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin`.
- If MSBuild cannot copy `WandEnhancer.exe` to `WandEnhancer\bin\Release`, check for a running `WandEnhancer` process started from that output path.
- Do not kill the user's running patcher without explicit user direction; report the lock holder and rerun after it closes.
