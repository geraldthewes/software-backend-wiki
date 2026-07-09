# XDG Base Directory Specification (Tier 2)

> **Tier 2** | Source: freedesktop.org basedir-spec 0.8 | Authority: established

## Summary

The XDG Base Directory Specification (freedesktop.org) defines where applications should look for and store user-specific files, by category: data, configuration, state, cache, and runtime files. Instead of scattering dotfiles and dot-directories across `$HOME` (`~/.myapp`, `~/.myapprc`), applications resolve a small set of environment variables — each with a well-defined default — and place each kind of file under the matching base directory.

For a coding agent, this specification is the default answer to "where should this CLI tool or service store its config/data/cache/state on Linux and other Unix-like systems?" Following it makes tools predictable for users (config is always under `~/.config`, caches are safe to delete under `~/.cache`), keeps backups and dotfile repos clean, and separates ephemeral files from persistent ones.

## Key Concepts

### Environment Variables and Defaults

| Variable | Contains | Default when unset or empty |
|----------|----------|-----------------------------|
| `$XDG_DATA_HOME` | User-specific data files | `$HOME/.local/share` |
| `$XDG_CONFIG_HOME` | User-specific configuration files | `$HOME/.config` |
| `$XDG_STATE_HOME` | User-specific state data (logs, history, layouts, undo state) | `$HOME/.local/state` |
| `$XDG_CACHE_HOME` | User-specific non-essential (cached) data | `$HOME/.cache` |
| `$XDG_RUNTIME_DIR` | User-specific runtime files (sockets, named pipes) | None — fall back with a warning |
| `$XDG_DATA_DIRS` | Preference-ordered search path for data files | `/usr/local/share/:/usr/share/` |
| `$XDG_CONFIG_DIRS` | Preference-ordered search path for configuration files | `/etc/xdg` |

User-specific executables go in `$HOME/.local/bin` (no variable; distributions should put it on `$PATH`).

### Path Validity

All paths set in these variables must be absolute. A relative path is invalid — treat the variable as unset and apply the default.

### Search Order and Precedence

- `$XDG_DATA_DIRS` and `$XDG_CONFIG_DIRS` are `$PATH`-style lists (colon-separated on Unix). The first directory listed is the most important; earlier entries override later ones.
- `$XDG_DATA_HOME` outranks every directory in `$XDG_DATA_DIRS`; `$XDG_CONFIG_HOME` outranks every directory in `$XDG_CONFIG_DIRS`. User files always win over system files.
- A full lookup of `subdir/filename` searches `$XDG_CONFIG_HOME/subdir/filename`, then each entry of `$XDG_CONFIG_DIRS` in order (same shape for data files). Any unset or empty variable contributes its default instead.

### Choosing the Right Category

| Category | Test | Examples |
|----------|------|----------|
| Config (`CONFIG_HOME`) | User-editable settings the application reads | config files, themes |
| Data (`DATA_HOME`) | Important, portable user data the application writes | databases, documents, plugins |
| State (`STATE_HOME`) | Persists across restarts but not important/portable enough for data | logs, history, recently-used lists, window layout, undo history |
| Cache (`CACHE_HOME`) | Non-essential; the application must survive its deletion | downloaded artifacts, compiled bytecode, thumbnails |
| Runtime (`RUNTIME_DIR`) | Lives only for the login session; communication/synchronization objects | sockets, named pipes, lock files |

### `$XDG_RUNTIME_DIR` Requirements

- MUST be owned by the user, who MUST be the only one with read/write access; Unix mode MUST be `0700`.
- Lifetime is bound to the login session: created at first login, removed at final logout; files MUST NOT survive reboot or a full logout/login cycle.
- MUST be on a local filesystem, not shared, and fully featured (AF_UNIX sockets, symlinks, hard links, file locking, sparse files, memory mapping, change notifications).
- Files MAY be periodically cleaned up: refresh a file's access timestamp at least every 6 hours or set its sticky bit to keep it.
- Do not place large files here — it may be backed by RAM. If unset, fall back to a replacement directory with similar capabilities and print a warning.

### Writing and Reading Rules

- When writing and the destination directory does not exist, create it with permission `0700`. If it already exists, do not change its permissions.
- Be prepared for writes to fail (directory could not be created, etc.) and handle it gracefully — an error message to the user is acceptable.
- When reading, if a file in one base directory is inaccessible (missing directory, missing file, no authorization), skip that directory and continue with the rest of the search path.
- A spec referencing `$XDG_DATA_DIRS`/`$XDG_CONFIG_DIRS` should define multi-directory behavior: use only the most important file, or define merge rules.

## Agent Guidance

### Do

- Store new tools' files under the XDG directories by category: config in `$XDG_CONFIG_HOME/<app>/`, data in `$XDG_DATA_HOME/<app>/`, state in `$XDG_STATE_HOME/<app>/`, cache in `$XDG_CACHE_HOME/<app>/`.
- Treat an unset, empty, or relative-path variable as unset and apply the spec default.
- Put logs, history, and recently-used lists in `$XDG_STATE_HOME` — they persist across restarts but are neither portable nor precious.
- Create missing destination directories with mode `0700` and leave existing directories' permissions untouched.
- Search configuration in precedence order (`$XDG_CONFIG_HOME` first, then `$XDG_CONFIG_DIRS` entries) and skip unreadable entries rather than aborting.
- Use existing helpers where available (e.g. Python `platformdirs`, Go `os.UserConfigDir`/`os.UserCacheDir`, `adrg/xdg`) instead of hand-rolling path resolution.
- Warn and fall back to a similar-capability directory when `$XDG_RUNTIME_DIR` is unset.

### Do Not

- Do not hardcode `~/.appname` or write new dotfiles directly into `$HOME` — that is exactly what this spec exists to prevent.
- Do not write to `$XDG_DATA_DIRS`/`$XDG_CONFIG_DIRS` locations (`/usr/share`, `/etc/xdg`) at runtime — those are install-time, system-level directories.
- Do not put caches in config or data directories — users and tooling assume `$XDG_CACHE_HOME` is safe to delete.
- Do not assume `$XDG_RUNTIME_DIR` is set, world-inaccessible files are unnecessary there, or that large files are acceptable in it.
- Do not store secrets or config in world-readable locations; the spec's `0700` directory-creation rule exists to protect user files.
- Do not `chmod` a pre-existing base directory to "fix" its permissions — the spec forbids changing permissions on existing directories.

## Checklist

- [ ] Every file the application writes is classified as config, data, state, cache, or runtime
- [ ] Each XDG variable is read from the environment; unset/empty/relative values fall back to the spec default
- [ ] No file is written directly into `$HOME` as a dotfile
- [ ] Config lookup searches `$XDG_CONFIG_HOME` then `$XDG_CONFIG_DIRS` in order (same for data)
- [ ] Missing destination directories are created with mode `0700`; existing permissions untouched
- [ ] Failed writes and unreadable search entries are handled gracefully (skip or clear error, never crash silently)
- [ ] Sockets/pipes go in `$XDG_RUNTIME_DIR` with a warned fallback when unset; no large files there
- [ ] Caches can be deleted at any time without breaking the application

## See Also

- wiki/tier2-core/twelve-factor-app/factors.md
- wiki/tier2-core/security-practices/overview.md
- wiki/tier1-sources/owasp/a05-security-misconfiguration.md
- wiki/tier3-working/python/logging.md
- wiki/tier3-working/golang/overview.md

## Source

Waldo Bastian, Allison Karlitskaya, Lennart Poettering, Johannes Löthberg. "XDG Base Directory Specification", Version 0.8, 8 May 2021. Freedesktop.org.
https://specifications.freedesktop.org/basedir/latest/ (raw capture: `references/xdg-basedir-spec-0.8.md`)
