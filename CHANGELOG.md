# Changelog

## 0.2.0

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
- Added `**/target/**`, `**/dist/**`, `**/build/**`, and `**/.direnv/**` to the
  default ignore globs.

## 0.1.0

- Phase 4 release pipeline and auto-download support.
