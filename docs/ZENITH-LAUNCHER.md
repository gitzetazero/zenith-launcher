# Zenith Launcher: what differs from XMCL

Zenith Launcher is a fork of **X Minecraft Launcher (XMCL)** by Voxelum (<https://github.com/Voxelum/x-minecraft-launcher>, MIT license).
XMCL's `LICENSE` and copyright are kept. Zenith's changes are concentrated and marked with a `// zenith:` comment, or live in new files
(`xmcl-runtime/zenith/`, `xmcl-electron-app/main/pluginZenith.ts`, `tools/zenith/`). The app keeps all of XMCL's languages; the few
new strings exist in English (default) and Spanish and fall back to English elsewhere.

## Base

| Item | Value |
|---|---|
| Upstream repository | `Voxelum/x-minecraft-launcher` (git remote `upstream`) |
| Base tag | `v0.73.1` |
| Base commit | `aeb524039` (`chore(release): version 0.73.1 (#1825)`) |
| Zenith Launcher version | `0.73.1-zenith.N` (`N` increases with each Zenith release) |

## What is different

- **Identity**: product name `Zenith Launcher`, app id `dev.zenith.launcher`, deep-link scheme `zenith-launcher://` (the internal `xmcl://` scheme is still used between modules),
  AppUserModelId `dev.zenith.launcher`, window titles, brand icons (`xmcl-electron-app/icons`, `xmcl-keystone-ui/src/assets/logo.webp`), default theme colors from the brand palette
  (`docs/branding/palette.json`; the user-selectable themes are kept).
- **Data**: the data folder is `%APPDATA%\zenith-launcher` (`LAUNCHER_NAME` in `xmcl-runtime/constant.ts`) and Chromium's user data goes to the same folder, so nothing mixes with XMCL's.
- **Packaging**: Windows x64 only: a per-user NSIS installer (no administrator) `Zenith-Launcher-Setup-x64.exe` and a portable `Zenith-Launcher-Portable-x64.exe`
  (`xmcl-electron-app/build/electron-builder.config.ts`). The Linux and macOS targets stay in the tree untouched but are not built or tested.
- **Texts**: occurrences of "XMCL" / "X Minecraft Launcher" used as the app name in `xmcl-keystone-ui/locales/*.yaml` become "Zenith Launcher" (not "XMCL Account", "Together", "Agent", etc., which name XMCL services).
  The About page carries the credit: *Zenith Launcher is based on X Minecraft Launcher (XMCL) by Voxelum, MIT license*.
- **Zenith integration** (`xmcl-electron-app/main/pluginZenith.ts`, `xmcl-runtime/zenith/instance.ts`): the Zenith bridge (`zenith-bridge-windows-x64.exe`) ships as an extra resource of the app. On startup, once the data folder is known and no instance named `Zenith` exists, the launcher
  1. copies the bridge as `java.exe` and `javaw.exe` into `%APPDATA%\zenith\java\bin` (or `ZENITH_HOME`), like `zenith-bridge setup` does;
  2. creates the `Zenith` instance (Minecraft `26.3`, `java` = that copy, Zenith icon) through XMCL's own `InstanceService.createInstance`.

  Why not run `zenith-bridge setup`: that command looks for the data folder in `%APPDATA%\xmcl\root`, which this launcher does not use. If you delete the instance on purpose it does not come back (marker file `zenith-instance.json`; delete it to recreate). The bridge handles the rest when you press Play (ownership check, signed game install, Java 25).
- **No XMCL services**: see the table below.

## What XMCL embedded and what Zenith Launcher does about it

| What XMCL embedded | What Zenith Launcher does | What the maintainer must provide |
|---|---|---|
| Microsoft client ID `1363d629-...` (`user/pluginOfficialUserApi.ts`, `xmclAccount/XmclAccountService.ts`) | **Removed.** Read from `ZENITH_MSA_CLIENT_ID` (environment), from `zenith-config.json` (`msaClientId`) or from the build (repository variable `ZENITH_MSA_CLIENT_ID`). Without it, Microsoft sign-in shows "Zenith Launcher needs an Azure app registered and approved by Mojang..." and never contacts Microsoft. | Their own Azure app, registered and approved by Mojang/Microsoft (personal accounts, public client, redirect `http://localhost:25555/auth`). |
| Application Insights keys (launcher, `xmcl-runtime-api/src/telemetry.ts`; resources, `infra/telemetry_resource.ts`) | **Removed.** Telemetry is off: the OpenTelemetry providers are created without exporters. | Nothing. |
| Hosted services `api.xmcl.app`, `ai.xmcl.app`, `signaling.xmcl.app`, `xmcl.app`, `xmcl-core-api.azurewebsites.net`, `xmcl.blob.core.windows.net` (flights, news, updates, ely.by authlib list, database mirror, XMCL account, built-in AI agent, P2P multiplayer, invite links) | **Removed.** The origins are `*.xmcl.disabled.invalid` (they never resolve) and a guard refuses such requests before they leave (`xmcl-runtime/zenith/blockedHosts.ts`, in the main process `fetch` and the renderer `https` handler). News are off, flights are never downloaded, mod databases come from GitHub only, ely.by uses its bundled list. Features that depended on those services (XMCL account, built-in agent, P2P/Together multiplayer) do not work. | Their own backend if those features are wanted (not planned). |
| CurseForge API key (XMCL's CI secret) | **No key.** `CURSEFORGE_API_KEY` (environment) or `curseforgeApiKey` in `zenith-config.json`. Without it CurseForge search does not work (Modrinth does). | Each user can set their own; the maintainer may add one from <https://console.curseforge.com/>. **There is no Settings field yet**: it is set by environment or file. |
| Modrinth OAuth app `GFz0B21y` and its code exchange at `api.xmcl.app/modrinth/auth` (`user/utils/loginModrinth.ts`) | **Removed.** `ZENITH_MODRINTH_CLIENT_ID`; without it Modrinth sign-in shows a message. A service holding the client secret is also needed (not included). Modrinth search and downloads without sign-in work. | Their own Modrinth OAuth app and code exchange. |
| LittleSkin OAuth app (`client_id` `393` and its **client_secret**, `littleSkin/LittleSkinUserService.ts`, `user/YggdrasilSeriveRegistry.ts`) | **Removed** (secret included). Optional `ZENITH_LITTLESKIN_CLIENT_ID`. Yggdrasil/LittleSkin accounts with user and password keep working. | Their own LittleSkin OAuth app if OIDC sign-in is wanted. |
| Discord application ID `1075044884400054363` (`presence/PresenceService.ts`) | **Removed.** `ZENITH_DISCORD_CLIENT_ID`; without it Discord presence does not connect (it is off by default). | Their own Discord application. |
| Updater (`api.xmcl.app/latest`, `xmcl.blob.core.windows.net`, Voxelum releases) | **Replaced.** It queries `api.github.com/repos/gitzetazero/zenith-launcher/releases` and, when a newer version exists, links to the Releases page (no automatic download, no `.asar`). | Publish releases in this repository. |
| Help links `xmcl.app/...` | Point to this document / the repository. | Nothing. |

Account flows not tied to an XMCL registration (Yggdrasil, ely.by, LittleSkin with password, offline) are untouched.

### Optional configuration (`%APPDATA%\zenith-launcher\zenith-config.json`)

```json
{ "msaClientId": "", "curseforgeApiKey": "", "modrinthClientId": "", "littleskinClientId": "", "discordClientId": "" }
```

The environment wins over the file, and the file over the build.

## Syncing with XMCL

```bash
git fetch upstream --tags
git merge v0.74.0        # or whichever upstream tag is next
```

Where conflicts will happen (everything else should merge cleanly):

- `README.md` (rewritten), `xmcl-electron-app/package.json` (name, version, author), `xmcl-electron-app/build/electron-builder.config.ts`.
- `xmcl-keystone-ui/locales/*.yaml` (the app-name replacement: take upstream's side and re-apply `XMCL` -> `Zenith Launcher` on the lines that name the app).
- `xmcl-runtime/user/pluginOfficialUserApi.ts`, `xmclAccount/XmclAccountService.ts`, `infra/plugins/telemetry.ts`, `infra/telemetry_resource.ts`, `infra/plugins/flights.ts`, `user/utils/loginModrinth.ts`, `presence/PresenceService.ts`, `littleSkin/*`, `app/pluginApiFallback.ts`: search for `zenith:`.
- `xmcl-electron-app/main/utils/updater.ts`, `ElectronLauncherApp.ts`, `definedPlugins.ts`, `xmcl-keystone-ui/src/views/SettingAbout.vue`, `composables/theme.ts`.
- The binary icons in `xmcl-electron-app/icons` and `logo.webp`.

After merging: `pnpm install`, `pnpm check`, `pnpm vitest run xmcl-runtime/zenith`, and make sure `grep -r "xmcl.app\|1363d629" xmcl-runtime xmcl-electron-app xmcl-keystone-ui/src` finds nothing new (apart from tests and credits).

## Build and releases

- `.github/workflows/zenith-release.yml` (manual, `windows-latest`, third-party actions pinned by commit SHA, no secrets; only `GITHUB_TOKEN` with `contents: write` at the publish step): installs, downloads and verifies the bridge, runs the unit tests of the touched code, builds with electron-builder, writes `SHA256SUMS` and publishes a **prerelease**.
- The bridge is verified by `tools/zenith/fetch-bridge.mjs`: the Ed25519 signature of `SHA256SUMS` from the `nightly` release of `gitzetazero/zenith` against Zenith's release key (the one in `zenith/bridge/zenith-release-keys.txt`), then the exe's SHA-256 against that signed list. Any mismatch fails the build.
- XMCL's inherited workflows (`build.yml`, `deploy-*.yml`, `sign-release.yml`, `e2e.yml`, ...) stay in the tree but are **disabled** in this repository (they need secrets or publish XMCL things).
- The exes are **not code-signed** (SmartScreen will warn).

## Limits

- Not verified on third-party PCs; builds and unit tests of the touched code run in CI and on a development PC.
- Microsoft sign-in does not work until an approved Azure app exists and is configured.
- Updates: notice and link to Releases only.
