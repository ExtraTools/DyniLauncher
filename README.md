# Dyni

A Windows desktop launcher for **Minecraft: Java Edition**, built with **Tauri 2 + Rust + React/TypeScript**. By **WaitDino**.

Vanilla / snapshot version management and launching — verified parallel downloads, offline & Microsoft accounts, multi-instance, and a hidden-console game process with live logs.

## Stack

- **Shell:** Tauri 2 (Rust backend + WebView2 UI)
- **Core (`crates/launcher-core`):** pure, unit-tested Rust — domain models, Mojang/Microsoft protocols, download engine, Java manager, launch builder
- **App (`src-tauri`):** thin Tauri layer — commands + IPC Channels
- **UI:** React 18 + TypeScript + Vite, lucide icons
- **Storage:** SQLite (`sqlx`); MS refresh tokens in the Windows Credential Manager (`keyring`)

## Features

- Real Mojang version list (24h-TTL + ETag cache), release/snapshot filter
- Verified (SHA1), parallel, atomic downloads of client jar + libraries + assets, with retry
- Mojang Java runtime auto-install per `javaVersion.component` (+ system-Java detection)
- Offline accounts (correct UUIDv3) and Microsoft accounts (device-code → Xbox → XSTS → Minecraft, entitlement-checked)
- True multi-instance: isolated game dirs over a shared content-addressed cache
- Launch builder: rules engine, placeholder substitution, `;`-classpath, Log4Shell mitigation, legacy `minecraftArguments` fallback
- `javaw` spawned with `CREATE_NO_WINDOW`; stdout/stderr streamed to an in-app log console
- NSIS installer + signed auto-update (`tauri-plugin-updater`)

## Develop

```powershell
pnpm install
pnpm tauri dev
```

Requires Rust (stable-msvc), the MSVC build tools, and Node + pnpm.

## Build the installer

```powershell
$env:TAURI_SIGNING_PRIVATE_KEY = (Get-Content src-tauri\.keys\updater.key -Raw)
$env:TAURI_SIGNING_PRIVATE_KEY_PASSWORD = ""
pnpm tauri build
```

Installer lands in `src-tauri/target/release/bundle/nsis/`.

## Configuration

- **Microsoft login** needs an Azure Entra app client id with the device-code flow + Minecraft API approval. Set `DYNI_AZURE_CLIENT_ID`; the default is a dev placeholder.
- **Auto-update** endpoints + signing: set the URL in `src-tauri/tauri.conf.json` (`plugins.updater.endpoints`) and keep the private key in `src-tauri/.keys/` (git-ignored).
- **Data dir:** `%APPDATA%\com.waitdino.dyni\` (private — not the shared `.minecraft`).

## Docs

- Design spec: `docs/superpowers/specs/2026-06-13-minecraft-launcher-design.md`
- Implementation plan: `docs/superpowers/plans/2026-06-13-minecraft-launcher-m0-m2.md`

> Unofficial launcher. Requires a legitimately owned copy of Minecraft. Not affiliated with Mojang or Microsoft.
