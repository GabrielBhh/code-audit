# Static Analysis Tools

Load in Step 3 (static analysis orchestration). For each detected language, check if tools are installed (`command -v <tool>`), run them, merge findings.

## Universal Fallback

```
1. command -v <tool>
2. If installed → run, parse output, merge findings
3. If not installed → try alternative
4. If nothing available → fall back to reasoning + note in report
```

**Never fail the audit if a tool is missing.** Missing tools = 🔵 Suggestion in "Tooling suggestions" section.

## Per-Language Tools

### Python
| Tool | Command | Catches |
|---|---|---|
| `ruff` | `ruff check .` | 800+ rules: style, bugs, security subset |
| `mypy` | `mypy .` | Type errors |
| `bandit` | `bandit -r .` | Security (hardcoded passwords, crypto, `shell=True`) |
| `semgrep` | `semgrep --config auto .` | Semantic patterns |
| `vulture` | `vulture .` | Dead code |

### JavaScript / TypeScript
| Tool | Command |
|---|---|
| `tsc --noEmit` | `npx tsc --noEmit` |
| `eslint` | `npx eslint .` |
| `biome` | `npx @biomejs/biome check .` |
| `semgrep` | `semgrep --config auto .` |
| `ts-prune` / `knip` | Dead exports |

### Go
| Tool | Command |
|---|---|
| `go vet` | `go vet ./...` |
| `staticcheck` | `staticcheck ./...` |
| `golangci-lint` | `golangci-lint run` |
| `gosec` | `gosec ./...` |

### Rust
| Tool | Command |
|---|---|
| `cargo check` | `cargo check --all-targets` |
| `cargo clippy` | `cargo clippy --all-targets` |
| `cargo-udeps` | `cargo +nightly udeps` |

### Ruby
| Tool | Command |
|---|---|
| `rubocop` | `bundle exec rubocop` |
| `brakeman` | `brakeman -A` |

### PHP
| Tool | Command |
|---|---|
| `phpstan` | `vendor/bin/phpstan analyse` |
| `psalm` | `vendor/bin/psalm` |

### Java / Kotlin
| Tool | Command |
|---|---|
| `spotbugs` | `mvn spotbugs:check` |
| `pmd` | `mvn pmd:check` |
| `detekt` (Kotlin) | `gradle detekt` |

### C / C++
| Tool | Command |
|---|---|
| `clang-tidy` | `clang-tidy src/*.cpp --` |
| `cppcheck` | `cppcheck --enable=all .` |

### Shell
| Tool | Command |
|---|---|
| `shellcheck` | `shellcheck *.sh` |

### Terraform / IaC
| Tool | Command |
|---|---|
| `tflint` | `tflint` |
| `tfsec` | `tfsec .` |
| `checkov` | `checkov -d .` |

### Docker
| Tool | Command |
|---|---|
| `hadolint` | `hadolint Dockerfile` |
| `trivy` | `trivy fs .` |

## Prefer Project's Own Scripts First

Before detecting globally, check:
- `package.json` `scripts` → run `npm run lint`
- `pyproject.toml` / `Makefile` → run `make lint`
- `justfile` → `just lint`

Uses the same rules the team's CI uses.

## Reporting

Every finding includes `detected-by: [tool1, tool2]` for confidence scoring. Multi-tool consensus = high confidence.
