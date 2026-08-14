# Changelog

## 0.2.0

- `mode` setting with `cmp-path` behavior: `"anywhere"` (default) completes
  bare path tokens (`src/ma`, `./fo`, `~/Do`, `/etc/ho`, `..`, `.`) in any
  document and outside string literals, so plain-text files work too.
  `"string"` keeps the string-literal-only behavior.

- Pathy is no longer Python-only. The sidecar LSP server now serves every
  language declared in `extension.toml` and completes paths inside string
  literals across languages.
- `context_gating` is generalized:
  - `"off"`: complete in any string literal.
  - `"smart"` (default): complete when the string contains an explicit path
    prefix (`./`, `../`, `/`, `~`, or Windows prefixes), or in Python
    call-site contexts (`open(...)`, `Path(...)`, ...).
  - `"strict"`: only complete in Python call-site contexts.
- `#` is only treated as a comment terminator in Python documents, so
  preprocessor lines such as `#include "./..."` are handled correctly.
- Request `workspace/configuration` right after initialize. `lsp-server` 0.7
  consumes the client's `initialized` notification inside `initialize_finish`,
  so the previous `initialized` handler never ran and settings only applied
  after the first `workspace/didChangeConfiguration`.
- Added `**/target/**`, `**/dist/**`, `**/build/**`, and `**/.direnv/**` to the
  default ignore globs.

## 0.1.0

- Phase 4 release pipeline and auto-download support.
