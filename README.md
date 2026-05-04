# GitHubFS – Total Commander WFX Plugin

Browse GitHub repositories directly inside Total Commander as a virtual filesystem.
Repositories, branches, directories and files appear as TC file entries.
Release assets are downloadable.
---

## Features

- Multiple repositories configurable via built-in dialog
- Each repo appears as a virtual folder at plugin root with a custom GitHub icon
- Optional **Default Branch**: skip branch list and show file contents directly
- **[Releases]** folder per repo (shown when GitHub releases exist)
  - Each release tag is a subfolder
  - `RELEASE_NOTES.md` virtual file with Markdown release notes
  - All release assets (`.zip`, `.tar.gz`, etc.) downloadable via F5
- Navigate branches → directories → files
- Copy files from GitHub to local disk (F5 / drag & drop) with progress bar
- Enter archives (`.zip`, `.7z`, `.rar`, …) with ENTER via TC packer plugin
- Delete configured repositories via DEL/F8 (removes from config, not from GitHub)
- Personal Access Tokens stored encrypted (Windows DPAPI)
- Get your token from https://github.com/settings/tokens
- **Global fallback token** for repos without an individual token
- Connection test per repository
- **F7 smart add**: type `owner/repo` to check GitHub and add in one step
- **[Quick add]** dialog for guided repo entry
- Custom icons: different icons for repos, [Configuration], and [Quick add]
- Release and file dates shown in TC file list (where available from GitHub API)
- DPI-aware configuration dialog, keyboard shortcuts (Ins/Del/F4/F8/Alt+Shift+F9)
- Dialogs centered on the TC window

---

## Requirements

| Item | Version |
|------|---------|
| Total Commander | 10.x 64-bit (recommended) or 32-bit |
| Windows | 7 / 10 / 11 |

---

## Installation

### Automatic (recommended)
Double-click `githubfs.wfx64` — TC detects `pluginst.inf` and installs automatically.

### Manual
1. In Total Commander: **Configuration → Options → Plugins → WFX Plugins → Add**
2. Select `githubfs.wfx64`
3. TC registers the plugin under *Network Neighborhood*

---

## Building from source

### With Lazarus IDE (recommended)

```bat
build-lazarus.bat 64      :: 64-bit -> githubfs.wfx64
build-lazarus.bat 32      :: 32-bit -> githubfs.wfx
build-lazarus.bat both    :: both architectures
```

Requires `lazbuild.exe` on PATH.

### Lazarus IDE
Open `githubfs.lpi`, select build mode **Release64**, press **Shift+F9**.

---

## Configuration

Double-click **[Configuration]** in the plugin root, or press F7 and type `owner/repo`.

The dialog shows the config file path at the top and lists all configured repositories.
A **Global Token** field provides a fallback for repos with no individual token.

### Repository fields

| Field | Description | Example |
|-------|-------------|---------|
| Display Name | Label shown in Total Commander | `my-project` or `hellouser/Hello-World` |
| GitHub Owner | Username or organisation | `hellouser` |
| Repository | Repo name without owner prefix | `Hello-World` |
| Access Token | Personal Access Token (optional) | `ghp_xxxx…` |
| Default Branch | Skip branch list, show files directly | `main` |
| Enabled | Uncheck to hide without deleting | ✓ |

Leave **Access Token** blank when editing to keep the existing token.

### F7 smart add

Press **F7** in the plugin root and type `owner/repo` (e.g. `hellouser/Hello-World`).
The plugin checks GitHub, adds the repo to config and refreshes the list automatically.
If the repo cannot be found on GitHub, the full Add dialog opens instead.

### Keyboard shortcuts in config dialog

| Key | Action |
|-----|--------|
| Insert | Add new repository |
| Del / F8 | Delete selected repository |
| F4 / Enter | Edit selected repository |
| Alt+Shift+F9 | Test connection |
| Double-click | Edit selected repository |

### Creating a GitHub Personal Access Token

1. GitHub → **Settings → Developer settings → Personal access tokens → Fine-grained tokens**
2. **Repository access**: select the repos you want to browse
3. **Permissions**: Contents → Read-only
4. Copy the token into the *Access Token* field (or the Global Token field)

> Tokens are stored **encrypted** using Windows DPAPI and are never written in plaintext.
> If DPAPI is unavailable, an XOR+Base64 fallback is used (low security, logged as warning).

---

## Virtual Path Hierarchy

```
\
+-- [Configuration]              opens config dialog  (gear icon)
+-- [Quick add]                  opens add-repo dialog (+ icon)
+-- hellouser/Hello-World          configured repo       (GitHub icon)
|   +-- [Releases]               present if GitHub releases exist
|   |   +-- v2.0.0\
|   |   |   +-- RELEASE_NOTES.md    Markdown release notes (virtual)
|   |   |   +-- Hello-World.zip     release asset (download with F5)
|   |   +-- v1.0.0\
|   |       +-- ...
|   +-- src\                     branch contents (with DefaultBranch=main)
|   +-- README.md
+-- my-org/my-repo
|   +-- main\                    branch (without DefaultBranch)
|   |   +-- lib\
|   |   +-- README.md
|   +-- develop\
+-- ...
```

---

## Archive navigation

Ctrl+PgDn on a `.zip`, `.7z`, `.rar`, `.tar.gz` (or any supported packer format) in a
GitHub repository will:
1. Download the archive via TC's built-in `FsGetFile` mechanism
2. Open it in the TC file panel using the configured WCX packer plugin

**Requirement:** `Configuration → Options → Packer → Treat archives like directories`
must be enabled (this is the default in a new TC installation).

---

## Config and log files

Both files are placed in the same folder as the plugin DLL.

| File | Purpose |
|------|---------|
| `githubfs.ini` | Encrypted repository list and global token |

---

## Source layout

```
githubfs.lpr          Library entry – WFX exports
githubfs.lpi          Lazarus project (BuildModes: Release64 / Release32)
build-lazarus.bat     Lazarus build script
pluginst.inf          TC auto-installer manifest
src/
  fsplugin.pas        TC WFX SDK v2.1 SE types
  GitHubTypes.pas     Data models (TRepoConfig, TReleaseInfo, TReleaseAsset, ...)
  GitHubAPI.pas       WinInet GitHub REST client (branches, contents, releases)
  GitHubCache.pas     In-memory cache (branches, contents, releases, assets)
  PluginConfig.pas    INI persistence + DPAPI token encryption
  VirtualFS.pas       Path parser + entry builder (all 6 path levels)
  uWfxMain.pas        All Fs* function implementations
  ConfigDlg.pas       Win32 config dialog (DPI-aware, keyboard shortcuts)
  IconProvider.pas    Custom 16x16 icons for repo, [Configuration], [Quick add]
  Logger.pas          TC log panel forwarding
  repo_icon_data.inc  Embedded ICO bytes (GitHub dark + Octocat)
  config_icon_data.inc Embedded ICO bytes (gear icon)
  quickadd_icon_data.inc Embedded ICO bytes (green + icon)
```

---

## GitHub API endpoints used

| Endpoint | Purpose |
|----------|---------|
| `GET /zen` | Reachability test (no auth) |
| `GET /user` | Token validity test |
| `GET /repos/{o}/{r}/branches` | List branches |
| `GET /repos/{o}/{r}/contents/{path}?ref={branch}` | Directory listing / file download |
| `GET /repos/{o}/{r}/releases` | List releases |
| `GET /repos/{o}/{r}/releases/tags/{tag}` | Release assets for a tag |
| `GET /repos/{o}/{r}` | Check if repo exists (F7 smart add) |

Files up to ~1 MB are returned base64-encoded inline.
Larger files use `download_url` with streaming and live progress bar.

---

## Caching

| Cached data | Cache key |
|-------------|-----------|
| Branch list | `owner|repo` |
| Directory contents | `owner|repo|branch|path` |
| Release list | `owner|repo` |
| Release assets | `owner|repo|tag|` |

The cache is cleared automatically when configuration is saved.

---

## Known limitations

- **Read-only**: file upload (`FsPutFile`) not implemented
- Branch listing limited to 100 branches per repo (GitHub API max per page)
- File modification dates: GitHub REST API does not return per-file dates.
  Only release assets show a date (`published_at`). Full per-file dates
  would require GraphQL (one call per directory listing).
- Non-ASCII branch/file names display as ANSI; paths remain functional
