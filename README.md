# GitHubFS – Total Commander WFX Plugin

Browse GitHub repositories directly inside Total Commander as a virtual filesystem.

---

## Features

- Multiple repositories configurable via built-in dialog
- Each repo appears as a folder at plugin root
- Optional **Default Branch**: skip the branch list and go directly to file contents
- Navigate branches → directories → files
- Copy files from GitHub to local disk (F5 / drag & drop)
- Progress bar during file download
- Personal Access Tokens stored encrypted (Windows DPAPI)
- Create token from https://github.com/settings/tokens
- **Global fallback token** for repos without their own token
- Connection test per repository
- **[Quick add]** shortcut for adding a repo and navigating to it immediately
- 404 errors shown as popup with details

---

## Requirements

| Item | Version |
|------|---------|
| Total Commander | 10.x 64-bit (x64) |
| Windows | 7 / 10 / 11 (64-bit) |
4
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
build-lazarus.bat 64
```

Requires `lazbuild.exe` on PATH. Produces `githubfs.wfx64` in the project root.

```bat
build-lazarus.bat 32    :: 32-bit build → githubfs.wfx
build-lazarus.bat both  :: both architectures
```

### With FPC command line

```bat
build.bat           :: 64-bit release
build.bat debug     :: 64-bit debug
```

### Lazarus IDE
Open `githubfs.lpi`, select build mode **Release64**, press **Shift+F9**.

---

## Configuration

Double-click **[Configuration]** in the plugin root.

The dialog shows the config file path at the top and lists all configured repositories.
A **Global Token** field at the top provides a fallback for repos that have no individual token.

### Adding or editing a repository

| Field | Description | Example |
|-------|-------------|---------|
| Display Name | Label shown in Total Commander | `my-project` |
| GitHub Owner | Username or organisation | `octocat` |
| Repository | Repo name without owner prefix | `Hello-World` |
| Access Token | Personal Access Token (PAT) | `ghp_xxxx…` |
| Default Branch | Optional: skip branch list, open this branch directly | `main` |
| Enabled | Uncheck to hide without deleting | ✓ |

Leave **Access Token** blank when editing to keep the existing token.

### [Quick add]

Double-click **[Quick add]** in the plugin root to open the Add dialog.
After clicking OK the plugin navigates directly into the new repository.

### Creating a GitHub Personal Access Token

1. GitHub → **Settings → Developer settings → Personal access tokens → Fine-grained tokens**
2. **Repository access**: select the repos you want to browse
3. **Permissions**: Contents → Read-only
4. Copy the token into the *Access Token* field

> Tokens are stored **encrypted** using Windows DPAPI and never written in plaintext.
> If DPAPI is unavailable, an XOR+Base64 fallback is used (less secure, logged).

---

## Virtual Path Hierarchy

### Without Default Branch

```
\
├── [Configuration]            ← opens config dialog
├── [Quick add]                ← add a repo and navigate to it
├── my-project                 ← configured repo
│   ├── main                   ← branch
│   │   ├── src/
│   │   └── README.md          ← file (copy with F5)
│   └── develop
│       └── ...
└── another-repo
    └── ...
```

### With Default Branch = "main"

```
\
├── my-project                 ← opens directly into "main" root
│   ├── src/
│   └── README.md
└── ...
```

---

## Config File

Both files are placed in the same folder as the plugin DLL (`githubfs.wfx64`).

| File | Purpose |
|------|---------|
| `githubfs.ini` | Encrypted repository list and global token |

---

## Source Layout

```
githubfs.lpr         Library entry point – WFX exports with name 'Fs*' clauses
githubfs.lpi         Lazarus project (BuildModes: Release64 / Release32)
build-lazarus.bat    Lazarus build script
build.bat            FPC command-line build script
pluginst.inf         TC auto-installer manifest
src/
  fsplugin.pas       TC WFX SDK types (official SDK v2.1 SE)
  GitHubTypes.pas    Data models: TRepoConfig, TBranchInfo, TContentEntry, TParsedPath
  GitHubAPI.pas      WinInet-based GitHub REST API client
  GitHubCache.pas    In-memory cache for branch lists and directory contents
  PluginConfig.pas   INI persistence + DPAPI token encryption + global token
  VirtualFS.pas      Path parser and entry builder (DefaultBranch-aware, cache-integrated)
  uWfxMain.pas       All Fs* function implementations
  ConfigDlg.pas      Win32 config dialog (DPI-aware, no RC file)
  Logger.pas         TC log panel forwarding
test/
  testfs.lpr         Minimal test plugin to verify build environment
  build_test.bat     Build script for test plugin
```

---

## Caching

Branch lists and directory contents are cached in memory for the duration of the TC session.
This eliminates repeated API calls when TC re-reads a directory (panel switch, F2 refresh, column info requests).

| Cached data | Cache key |
|---|---|
| Branch list | `owner\|repo` |
| Directory contents | `owner\|repo\|branch\|path` |

The cache is cleared automatically when the configuration is saved, ensuring stale data is never shown after a repo is added, edited, or removed.
File contents themselves are not cached — TC handles local copies.

---



| Endpoint | Purpose |
|----------|---------|
| `GET /zen` | Network reachability test (no auth required) |
| `GET /user` | Token validity test (requires auth) |
| `GET /repos/{o}/{r}/branches?per_page=100` | List branches |
| `GET /repos/{o}/{r}/contents/{path}?ref={branch}` | List directory / get file |

Files ≤ ~1 MB are returned base64-encoded inline.
Larger files use the `download_url` with streaming download and live progress bar.

---

## Known Limitations

- **Read-only**: file upload (`FsPutFile`) is not implemented
- Branch listing is limited to 100 branches (GitHub API maximum per page)
- No caching: every directory listing triggers a live API call
- ANSI paths only (Unicode branch/file names with non-ASCII characters may not display correctly)
