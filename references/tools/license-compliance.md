# License Compliance

Load when running license scanning. Flag copyleft contamination and unknown licenses.

## Tools

| Ecosystem | Tool | Command |
|---|---|---|
| Python | `pip-licenses` | `pip-licenses --format=json` |
| JS/TS | `license-checker` | `npx license-checker --json` |
| Rust | `cargo-license` | `cargo license` |
| Go | `go-licenses` | `go-licenses report ./...` |
| Ruby | `license_finder` | `license_finder` |
| Java | Maven license plugin | `mvn license:add-third-party` |
| Multi | `scancode-toolkit` | `scancode-toolkit -ilp --json-pp out.json .` |

## Process

1. Read project license from `LICENSE` / `LICENSE.md`
2. Run tool above to list transitive dep licenses
3. Flag conflicts

## Conflicts to Flag

| Project license | Transitive dep license | Severity |
|---|---|---|
| MIT/Apache/BSD | GPL, AGPL, LGPL | 🔴 Critical — copyleft contamination |
| MIT | GPL used as library | 🔴 — distribution triggers GPL requirements |
| Any | Unknown / Custom / unrecognized | 🟡 Warning |
| Any | "Non-commercial use only" clause | 🟡 Warning |

If no tool is installed, emit a 🔵 Suggestion with the install command.
