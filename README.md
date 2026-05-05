# GitHubFS – Total Commander WFX Plugin

Browse GitHub repositories directly inside Total Commander as a virtual filesystem.
Repositories, branches, directories and files appear as TC file entries.
Release assets are downloadable. Archives can be extracted and browsed in-place.

---

## Features

- Multiple repositories configurable via built-in dialog
- Each repo appears as a virtual folder at plugin root with a **user avatar icon**
- Optional **Default Branch**: skip branch list and show file contents directly
- **[Releases]** folder per repo (shown when GitHub releases exist)
  - Each release tag is a subfolder
  - `RELEASE_NOTES.md` virtual file with Markdown release notes
  - All release assets downloadable via F5
- Navigate branches → directories → files
- Copy files from GitHub to local disk (F5 / drag & drop) with progress bar
- **Archive navigation**: ENTER on `.zip`, `.7z`, `.rar`, `.tar.gz`, etc. downloads,
  extracts to temp folder and enters it directly – navigate subdirs freely,
  copy files with F5; extracted files deleted automatically on leaving the archive
- **File and directory sizes** shown in TC file list (via Git Tree API, configurable)
  - Alt+Enter on a branch or directory shows recursive size regardless of setting
- Delete configured repositories via DEL/F8 (removes from config, not from GitHub)
- Personal Access Tokens stored encrypted (Windows DPAPI)
- **Global fallback token** for repos without an individual token
- Connection test per repository (button + Alt+Shift+F9)
- **F7 smart add** supports three formats:
  - Plain name → opens Add dialog
  - `owner/repo` → checks GitHub and adds in one step
  - Full GitHub URL (e.g. `https://github.com/owner/repo/tree/branch`) → parses
    owner, repo and branch, adds immediately
- **[Quick add]** dialog for guided repo entry with URL auto-fill
- **Add Repository dialog** with "GitHub URL:" field at top – paste any GitHub URL
  and all fields fill automatically
- Custom icons: user avatar per repo (downloaded from GitHub), gear icon for
  [Configuration], green + for [Quick add]
- Release and file dates shown in TC file list (where available from GitHub API)
- DPI-aware configuration dialog, full keyboard shortcuts
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

Double-click **[Configuration]** in the plugin root, or press Alt+Enter on any
repo entry, or press F7 and type `owner/repo` or a GitHub URL.

The dialog shows the config file path at the top and lists all configured repositories.
A **Global Token** field provides a fallback for repos with no individual token.
The **"Show file and directory sizes"** checkbox enables Git Tree API size fetching.

### Repository fields

| Field | Description | Example |
|-------|-------------|---------|
| GitHub URL | Paste any GitHub URL to auto-fill all fields | `https://github.com/owner/repo` |
| Display Name | Label shown in Total Commander | `my-project` or `hellouser/Hello-World` |
| GitHub Owner | Username or organisation | `hellouser` |
| Repository | Repo name without owner prefix | `Hello-World` |
| Access Token | Personal Access Token (optional for public repos) | `ghp_xxxx…` |
| Default Branch | Skip branch list, show files directly | `main` |
| Enabled | Uncheck to hide without deleting | ✓ |

Leave **Access Token** blank when editing to keep the existing token.

The **Refresh Avatar** button re-downloads the owner's avatar from GitHub
and updates the repo icon.

### F7 smart add

Press **F7** in the plugin root and type any of:

| Input | Action |
|-------|--------|
| `my-repo` | Opens Add dialog with name pre-filled |
| `octocat/Hello-World` | Checks GitHub → adds immediately |
| `https://github.com/octocat/Hello-World/tree/main` | Parses owner, repo, branch → adds immediately |

If the repo cannot be found on GitHub the full Add dialog opens instead.

### Keyboard shortcuts in config dialog

| Key | Action |
|-----|--------|
| Insert | Add new repository |
| Del / F8 | Delete selected repository |
| F4 / Enter | Edit selected repository |
| Alt+Shift+F9 | Test connection for selected repo |
| Double-click | Edit selected repository |
| ESC | Close dialog (= Cancel) |

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
+-- hellouser/Hello-World          configured repo       (user avatar icon)
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
|   |   +-- archive.zip          press ENTER to extract and browse
|   +-- develop\
+-- ...
```

---

## Archive navigation

ENTER on a `.zip`, `.7z`, `.rar`, `.tar.gz` (or other supported packer format):

1. The archive is downloaded to a temp folder (`%TEMP%\githubfs\`)
2. The archive is extracted to a subfolder using Windows Shell automation
3. TC navigates into the extracted folder – all files and subdirs are accessible
4. Extracted files can be copied with F5 like any local file
5. When you navigate back (`..`), the extracted folder **and** the downloaded
   archive are deleted automatically

---

## User Avatar Icons

Each repository displays the avatar of its GitHub owner as its icon in TC.

- Avatars are **downloaded automatically** when a repo is added (requires token or
  the user being public)
- Stored as Base64-encoded PNG in `githubfs.ini` (64×64 px, compressed)
- The **Refresh Avatar** button in the Edit dialog re-downloads the current avatar
- Avatars are rendered using GDI+ (`gdiplus.dll`) – supports PNG and JPEG
- If no avatar is available, the default GitHub icon is shown

---

## File and Directory Sizes (Git Tree API)

When **"Show file and directory sizes"** is enabled (default: on):

- On first navigation into a branch, the Git Tree API is called once:
  `GET /repos/{o}/{r}/git/trees/{branch}?recursive=1`
- The response contains all file paths and sizes for the entire branch
- Directory sizes are computed by summing all files under each directory path
- Results are cached per branch – subsequent navigations are instant
- **Requires a token** (anonymous API calls have very low rate limits)
- **Alt+Enter** on any directory always shows its recursive size, regardless
  of the global setting

---

## Config file

The INI file is placed in the same folder as the plugin DLL.

| File | Purpose |
|------|---------|
| `githubfs.ini` | Repository list, encrypted tokens, avatar images, settings |

---

## Source layout

```
githubfs.lpr            Library entry – WFX exports
githubfs.lpi            Lazarus project (BuildModes: Release64 / Release32)
build-lazarus.bat       Lazarus build script
pluginst.inf            TC auto-installer manifest
src/
  fsplugin.pas          TC WFX SDK v2.1 SE types
  GitHubTypes.pas       Data models (TRepoConfig, TTreeEntry, TVFSEntry, …)
  GitHubAPI.pas         WinInet GitHub REST client + GHFetchAvatar + GHGetTree
  GitHubCache.pas       In-memory cache (branches, contents, releases, tree)
  PluginConfig.pas      INI persistence + DPAPI token encryption
  VirtualFS.pas         Path parser + entry builder + SumTreeDirSize
  uWfxMain.pas          All Fs* function implementations
  ConfigDlg.pas         Win32 config dialog (DPI-aware, keyboard shortcuts)
  IconProvider.pas      GDI+-based avatar loader + embedded fallback icons
  Logger.pas            TC log panel forwarding
  repo_icon_data.inc    Embedded ICO bytes (fallback repo icon)
  config_icon_data.inc  Embedded ICO bytes (gear icon)
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
| `GET /repos/{o}/{r}/git/trees/{ref}?recursive=1` | Full file tree with sizes |
| `GET /users/{owner}` | User profile (avatar_url) |
| `GET {avatar_url}?s=64` | User avatar image download |

Files up to ~1 MB are returned base64-encoded inline.
Larger files use `download_url` with streaming and live progress bar.

---

## Caching

| Cached data | Cache key |
|-------------|-----------|
| Branch list | `owner\|repo` |
| Directory contents | `owner\|repo\|branch\|path` |
| Release list | `owner\|repo` |
| Release assets | `owner\|repo\|tag` |
| Git file tree | `owner\|repo\|branch` |

The cache is cleared automatically when configuration is saved.

---

## Known limitations

- **Read-only**: file upload (`FsPutFile`) not implemented
- Branch listing limited to 100 branches per repo (GitHub API max per page)
- File modification dates: GitHub REST API does not return per-file dates.
  Only release assets show a date (`published_at`). Full per-file dates
- Archive extraction supports one level only (entering an archive inside an
  extracted archive is not supported)
