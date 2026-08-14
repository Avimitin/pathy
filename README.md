# Pathy

Pathy is a Zed extension (Rust -> WASM) that launches a sidecar LSP server to
provide filesystem path completions inside string literals. It runs as a
secondary language server and only advertises completion capability.

It completes relative (`./`, `../`), absolute (`/`), and home (`~/`) paths, in
any language listed in `extension.toml`, and merges with the completions of the
language's primary server.

By default it behaves like nvim-cmp's `cmp-path`: any path-like token
(`src/ma`, `./fo`, `~/Do`, `/etc/ho`, `..`) is completed regardless of the
surrounding context or language — including plain-text files. Inside string
literals it additionally understands quoting and Python call-site contexts.

## Install (dev)

1) Build the extension (Zed builds WASM automatically) or open the repo in Zed.
2) In Zed, open the command palette and select "Extensions: Install Dev Extension".
3) Choose this repo root (the directory containing `extension.toml`).
4) Grant required capabilities if prompted (see below).

## Required capabilities

The extension needs:
- `process:exec` to launch the sidecar server
- `download_file` to fetch release binaries

These are already declared in `extension.toml`. If Zed prompts for approval,
accept them.

## Configuration

Settings live under `lsp.pathy.settings` in your Zed settings. This single block
contains both extension-level settings (download behavior) and server settings
(completion behavior).

### Language server ordering

Enable Pathy per language, keeping the primary language server first:

```json
{
  "languages": {
    "Python": { "language_servers": ["pyright", "pathy", "..."] },
    "Rust": { "language_servers": ["rust-analyzer", "pathy", "..."] }
  }
}
```

For languages without an explicit `language_servers` list, Pathy is started
automatically once the language is listed in `extension.toml`.

### Extension settings (download/runtime)

```json
{
  "lsp": {
    "pathy": {
      "settings": {
        "auto_download": true,
        "server_path": null,
        "release_channel": "stable",
        "base_url": null,
        "verify_checksum": true,
        "cache_dir": null
      }
    }
  }
}
```

Notes:
- `server_path` takes precedence. If set and exists, no download is attempted.
- `auto_download` must be `true` to fetch release assets.
- `release_channel` only supports `stable`.
- `base_url` is advanced; default points at `https://github.com/Avimitin/pathy`.
- `cache_dir` must be a relative path (relative to the extension working dir).

### Server settings (completion behavior)

Defaults shown in parentheses:

- `enable` (true)
- `mode` ("anywhere"): "anywhere" | "string"
- `path_prefix_fallback` (true)
- `context_gating` ("smart"): "off" | "smart" | "strict"
- `base_dir` ("file_dir"): "file_dir" | "workspace_root" | "both"
- `workspace_root_strategy` ("lsp_root_uri"): "lsp_root_uri" | "disabled"
- `max_results` (80)
- `show_hidden` (false)
- `include_files` (true)
- `include_directories` (true)
- `directory_trailing_slash` (true)
- `ignore_globs` (["**/.git/**", "**/.venv/**", "**/venv/**", "**/__pycache__/**",
  "**/.pytest_cache/**", "**/.mypy_cache/**", "**/.ruff_cache/**",
  "**/node_modules/**", "**/target/**", "**/dist/**", "**/build/**", "**/.direnv/**"])
- `prefer_forward_slashes` (true)
- `expand_tilde` (true)
- `windows_enable_drive_prefix` (true)
- `windows_enable_unc` (true)
- `cache_ttl_ms` (500)
- `cache_max_dirs` (64)
- `stat_strategy` ("lazy"): "none" | "lazy" | "eager"

`mode` semantics:

- `"anywhere"` (default): complete bare path tokens anywhere in the document,
  outside string literals too (`cmp-path` behavior). This covers plain-text
  files, code, comments, and everything in between.
- `"string"`: only complete inside string literals.

`context_gating` applies to string-literal completion:

- `"off"`: complete anywhere.
- `"smart"`: complete on an explicit path prefix, or in Python call-site
  contexts (`open(...)`, `Path(...)`, `read_csv(...)`, `path=` arguments, ...).
- `"strict"`: only complete in Python call-site contexts.

Example override:

```json
{
  "lsp": {
    "pathy": {
      "settings": {
        "max_results": 40,
        "show_hidden": true,
        "mode": "anywhere"
      }
    }
  }
}
```

## Local testing

To use a local build of the server:

```bash
cd server
cargo build --release
```

Then set `server_path` to the built binary:
- macOS/Linux: `server/target/release/pathy-server`
- Windows: `server/target/release/pathy-server.exe`

## Release assets

Assets are raw binaries (no archives):
- `pathy-server_<VERSION>_macos_x86_64`
- `pathy-server_<VERSION>_macos_aarch64`
- `pathy-server_<VERSION>_linux_x86_64`
- `pathy-server_<VERSION>_windows_x86_64.exe`
- `checksums-<VERSION>.txt`

## License

MIT. See `LICENSE`.
