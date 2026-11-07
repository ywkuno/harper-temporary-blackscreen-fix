# Harper v3.0.0 — experimental Windows black-screen source patch

**Status: source patch only. Not a Windows installer. Not compiled or tested on Windows here.**

This package isolates the overlay rendering portion of [Automattic/harper PR #4447](https://github.com/Automattic/harper/pull/4447). The PR was open as of 7 November 2026. It addresses black/white screens on Windows when Harper's system-wide service starts. See [issue #4417](https://github.com/Automattic/harper/issues/4417) and [issue #4578](https://github.com/Automattic/harper/issues/4578).

The patch is targeted **specifically at official Harper v3.0.0 source** and changes only `harper-desktop/src-tauri/src/highlighter/window.rs`. Unlike PR #4447, it does not include unrelated fixes for the tray, suggestion replacement or UI Automation.

## What changes?

1. The always-on-top Windows overlay requests `WS_EX_NOREDIRECTIONBITMAP`, and disables the taskbar entry and shadow.
2. Its `wgpu` backend explicitly chooses DirectX 12 plus DirectComposition (`DxgiFromVisual`), which can support premultiplied alpha transparency instead of an opaque black HWND surface.
3. Other platforms' rendering backend behavior stays unchanged.

This source-level fix is based on the publicly reported solution and **is not independently verified** here. It could fail on a specific GPU, device or Windows build. It may still have unrelated bugs.

## Apply to source (for a developer)

Install Git for Windows. In PowerShell:

```powershell
git clone --depth 1 --branch v3.0.0 https://github.com/Automattic/harper.git harper-v3
.\APPLY-PATCH.ps1 -RepoPath .\harper-v3
```

If scripts are restricted by policy, do not disable security settings. Apply the patch directly via Git:

```powershell
git -C .\harper-v3 apply --check .\harper-v3.0.0-windows-overlay.patch
git -C .\harper-v3 apply .\harper-v3.0.0-windows-overlay.patch
```

## Build an unsigned Windows installer

This is a Rust / Tauri / Node / WASM monorepo, **not a stand-alone .rs program**. Native building needs Microsoft C++ Build Tools, Rust/MSVC target, Node.js, pnpm, `just`, `wasm-pack`, a Unix-compatible bash/perl environment, and Tauri build dependencies. For the exact build pipeline, consult the official [Harper Desktop contributor documentation](https://writewithharper.com/docs/contributors/desktop) and the project's `justfile`.

For convenience, **`build-patched-harper.yml`** is an experimental manual GitHub Actions workflow. Add it to `.github/workflows/build-patched-harper.yml` in a separate repository or fork under your own GitHub account, and commit `harper-v3.0.0-windows-overlay.patch` at the repository root. Run the workflow manually and inspect the logs/artifacts. It uses the official v3.0.0 tag. The build is **unsigned**, unlike Automattic's signed official releases; it should be tested in a disposable Windows VM first. Do not disable Windows security features to install it.

## Critical safety notes

- **Do not send this `.patch` file to a nontechnical user expecting it to install.** It is source code, not a binary hotfix.
- Don't overwrite the original Harper installation or replace executable files blindly.
- Prefer the Harper browser extension or Word's Editor for time-sensitive coursework until a tested, signed release fixes the issue.
- If testing a custom Windows build, disable auto-start, close Harper first, and keep the taskkill recovery command ready: `taskkill /F /IM harper-desktop.exe /T`.
- This patch addresses desktop overlay blackouts; it does **not** fix Harper's lack of integration with Microsoft Word in SharePoint in a browser.

## Provenance

- Base source file: [`Automattic/harper@v3.0.0`](https://github.com/Automattic/harper/blob/v3.0.0/harper-desktop/src-tauri/src/highlighter/window.rs)
- Upstream community approach: [PR #4447](https://github.com/Automattic/harper/pull/4447)
- Packaged here as a minimal, experimental one-file change.
