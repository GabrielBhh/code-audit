---
name: code-audit
description: |
  Deep local code audit — runs the project's real analyzers (ruff, eslint, gosec,
  cargo clippy, semgrep, bandit, etc.), scans dependencies for CVEs via OSV.dev,
  detects secrets in git history with gitleaks, analyses git-history hotspots
  (churn × complexity, ownership, temporal coupling), cross-checks findings
  against project conventions (AGENTS.md, CONTRIBUTING.md, ADRs), and layers
  reasoning on top of tool output. Respects `.codereview-baseline.json` so only
  new regressions surface. Trigger when the user asks to "audit my code",
  "review my changes", "check for security issues", "run a deep review",
  "run the tools", or mentions /code-audit. Position: the gate before
  commit/PR — deeper than in-flow reasoning reviews. Languages: Python, JS/TS,
  Go, Java, Kotlin, Ruby, Rust, C, C++, PHP, C#, Dart/Flutter, SQL, Shell,
  Swift, Dockerfile, YAML, Terraform.
---

# code-audit

A deep, **tool-orchestrating** code audit. Runs the real static analyzers, CVE scanners, secret history scanners, and git-intelligence commands on your local repo, then merges their output with Claude's reasoning into one structured report.

**Positioning**: this is the **gate** — the deep pass you run before commit/PR. It's intentionally different from Claude's built-in review (which is the fast in-flow first-pass). This skill adds what raw prompting cannot: real tool execution, CVE data from OSV.dev, secrets in git history, and git-archaeology.

---

## Pipeline overview

Every invocation runs these steps in order:

| Step | What it does |
|---|---|
| 1 | Resolve target + load config (baseline, ignore, conventions) |
| 2 | Claude reasoning pass (baked-in first-layer review) |
| 3 | Static analysis orchestration (per-language tools) |
| 4 | CVE + license scanning (OSV.dev + CLI tools) |
| 5 | Secrets in git history (gitleaks/trufflehog) |
| 6 | Git intelligence (hotspots, ownership, coupling) |
| 7 | Convention compliance (CLAUDE.md, AGENTS.md, ADRs) |
| 8 | Merge + dedupe + apply baseline + ignore |
| 9 | Write structured report |

Each tool step is **opportunistic** — use what's installed, fall back to Claude reasoning if nothing is available, and note which tools would have added value.

---

## Step 1 — Resolve target and load config

### Resolve what to audit

| User input | How to gather the code |
|---|---|
| (nothing / "my changes") | `git diff HEAD` |
| `staged` | `git diff --cached` |
| `unstaged` | `git diff` |
| `all` / "whole codebase" / "everything" | `git ls-files` |
| `<directory-path>` (e.g. `src/`) | `git ls-files <dir>` |
| `<file-path>` | Read the file directly |
| `<commit-hash>` | `git show <hash>` |
| `<branch>` | `git diff main..<branch>` |
| `<hash1>..<hash2>` | `git diff <hash1>..<hash2>` |
| `baseline update` | Re-snapshot current findings as accepted baseline |

### Load project config (conditional)

Check for these files with `ls`. Only load the reference files if the corresponding project files exist — don't load references unconditionally.

| Project file present | Reference file to load |
|---|---|
| `.codereview-baseline.json` | `references/baseline-and-state.md` |
| Any of: `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, `docs/adr/*.md` | `references/convention-compliance-rules.md` (in Step 7) |

Also read these directly if present (no reference file needed):
- `.codereview-ignore` — patterns and file-specific ignores (follow the format described in baseline-and-state.md only if loaded)
- Tool configs: `.editorconfig`, `.prettierrc`, `.eslintrc`, `ruff.toml`, etc. (use contents directly)

### Large-codebase strategy

For `all` or directory reviews of >50 files: summarise scope up front, then work through by language group. For >150 files: ask whether to narrow scope before starting.

---

## Step 2 — Reasoning pass (baked-in first layer)

Apply baseline reasoning before running tools. This is the skill's equivalent of an in-flow review — included here so users don't need to invoke two separate skills.

### Lazy loading (IMPORTANT for token efficiency)

Reference files are split for lazy loading. **Load only what applies to the files in scope.** Never load all reference files up front.

#### Language files — load per detected language

Always load `references/lang/_common.md` (language-agnostic simplification patterns, ~300 tokens).

Then for each detected language, load ONLY that language's file:

| Extension | File to load |
|---|---|
| `.py` | `references/lang/python.md` |
| `.js`, `.ts`, `.jsx`, `.tsx` | `references/lang/javascript-typescript.md` |
| `.go` | `references/lang/go.md` |
| `.rs` | `references/lang/rust.md` |
| `.rb` | `references/lang/ruby.md` |
| `.php` | `references/lang/php.md` |
| `.java`, `.kt` | `references/lang/java-kotlin.md` |
| `.cs` | `references/lang/csharp.md` |
| `.dart` | `references/lang/dart-flutter.md` |
| `.c`, `.cpp`, `.h`, `.hpp` | `references/lang/c-cpp.md` |
| `.swift` | `references/lang/swift.md` |
| `.sql` | `references/lang/sql.md` |
| `.sh`, `.bash` | `references/lang/shell.md` |
| `Dockerfile`, `*.tf`, `*.yaml` (k8s) | `references/lang/dockerfile-iac.md` |

#### Security files — load by applicability

- **Always load** `references/security/core-owasp.md` (universal OWASP concerns, ~700 tokens)
- **Web languages** (JS/TS/Python/PHP/Ruby/Java/Kotlin/Go in web contexts): load `references/security/web-patterns.md`
- **Auth code detected** (JWT, OAuth, session handling): load `references/security/api-auth.md`
- **Native languages** (C, C++, and `unsafe` Rust): load `references/security/native-patterns.md`
- `references/security/secrets-regex.md` — load ONLY in Step 5 (secret scanning)

#### Other conditional references

- `references/api-design-rules.md` — load only if REST controllers, GraphQL schemas, or `.proto` files detected
- `references/convention-compliance-rules.md` — load only if any convention doc exists in the repo (see Step 7)
- `references/baseline-and-state.md` — load only if `.codereview-baseline.json` exists or user invokes `baseline update`
- `references/git-intelligence-rules.md` — load only if scope is `all` or `<branch>` (skip for small `staged` diffs)

**Do not load what isn't needed.** On a small Python staged diff, total reference load is typically ~2-3K tokens, not 15K.

### What to look for in this step

Using the loaded references, focus on:

- **Bugs & logic errors** — null/undefined dereferences, off-by-one, race conditions, unhandled edge cases, wrong conditionals, crash paths
- **Security patterns** — OWASP findings per the loaded `references/security/*.md` files
- **Non-obvious language patterns** — per the loaded `references/lang/*.md` file(s)
- **API design** — per `references/api-design-rules.md` if loaded

**Important**: reference files are intentionally curated. Do not invent generic "best practice" findings beyond what tools or references explicitly call out. The value of this skill is tool execution + repo-wide analysis, not repeating what reasoning would produce by default.

---

## Step 3 — Static analysis orchestration

Load `references/tools/static-analysis.md`. For each detected language:

1. Detect which tools are installed (`command -v <tool>`)
2. Run the installed tools
3. Parse output, map findings to severity tiers
4. Note which tools were NOT installed and would have added value

**Fallback**: if no tools are installed for an ecosystem, rely on Step 2 reasoning and emit a single 🔵 Suggestion noting which tools would help.

---

## Step 4 — CVE + license scanning

Only run if dependency files exist in the repo.

Load `references/tools/cve-scanning.md` for CVE process and ecosystem map. If running license checks, also load `references/tools/license-compliance.md`.

1. Try the ecosystem CLI tool first (`pip-audit`, `npm audit`, `cargo audit`, `govulncheck`, `bundle audit`, `composer audit`, `dotnet list package --vulnerable`, `dart pub outdated`, `trivy fs .`)
2. If the tool is not installed, POST to `https://api.osv.dev/v1/querybatch` with extracted `(name, version)` pairs
3. Map CVSS scores to severity: ≥7.0 → 🔴, 4.0–6.9 → 🟡, <4.0 → 🔵
4. For license compliance, run `pip-licenses` / `license-checker` / `cargo-license` and flag copyleft contamination

---

## Step 5 — Secrets in git history

Load `references/tools/secret-history.md` and `references/security/secrets-regex.md`.

Try `gitleaks detect --source . --report-format json --no-banner`. If not available, try `trufflehog git file://. --json`. If neither is installed, emit a single 🔵 Suggestion with the install command. Do not scan history manually.

Finding severity: **always 🔴 Critical** when a secret is detected, even if later deleted from HEAD — the commit SHA is public the moment it was pushed.

---

## Step 6 — Git intelligence

**Skip this step entirely for small-scope reviews** (single file, small `staged` diff). Only run when:
- Scope is `all` or `<branch>`
- Or the `staged` diff touches >5 files

If the step runs, load `references/git-intelligence-rules.md`. Analyze:

- **Churn × complexity hotspots**: flag files frequently changed AND complex (🟡)
- **Ownership / bus factor**: flag single-author files in the diff (🔵, or 🟡 for core infra)
- **Temporal coupling**: files changing together across modules >70% (🔵)
- **Dead file candidates**: only on `all` reviews — tracked files with zero commits in 2+ years (🔵)

---

## Step 7 — Convention compliance

**Skip this step entirely if no convention docs exist in the repo.** Quick check first: `ls CLAUDE.md AGENTS.md CONTRIBUTING.md ARCHITECTURE.md docs/adr/ 2>/dev/null`. If none of those exist, move to Step 8.

If convention docs do exist, load `references/convention-compliance-rules.md`. Cross-check findings against:

- Rules stated in `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`
- Architectural decisions in `docs/adr/*.md` (only those with status `Accepted`)
- Declared conventions in `.editorconfig`, linter configs, `package.json` scripts

When flagging a violation: **cite the source document and line**, e.g. `violates ADR-012 (docs/adr/0012-use-postgres.md:L14)`. This turns generic findings into team-specific guardrails.

---

## Step 8 — Merge, dedupe, apply baseline

1. **Dedupe**: the same file:line flagged by multiple tools → single finding with `detected-by: [tool1, tool2, claude]` for confidence scoring
2. **Apply baseline**: if `.codereview-baseline.json` exists, suppress findings that match. Only surface regressions and new findings.
3. **Apply ignore**: respect `.codereview-ignore` patterns (glob-style file patterns + per-line `# codereview: ignore` comments)
4. **Confidence scoring**:
   - Flagged by 2+ tools + Claude = **high confidence** (bump severity up a tier if borderline)
   - Flagged by 1 tool only = **medium**
   - Claude reasoning only = **low** (but can still be 🔴 if clearly a bug)

---

## Step 9 — Write the report

```
# Code Audit Report
**Target**: <what was audited>
**Date**: <today's date>
**Language(s)**: <detected>
**Scope**: <N files / N lines>
**Tools run**: <list>
**Tools missing**: <list>
**Baseline**: <in-use / not present>

## Summary

| Severity | Count (new) | Count (baseline) |
|---|---|---|
| 🔴 Critical | N | M |
| 🟡 Warning | N | M |
| 🔵 Suggestion | N | M |

## Findings

### 🔴 CRITICAL — [Category]
**`path/to/file.ext:42`**  •  detected-by: `gosec` + Claude (high confidence)
One-sentence description.
**Fix**: concrete snippet or short explanation.

### 🟡 WARNING — [Category]
**`path/to/file.ext:17`**  •  detected-by: `ruff` (medium confidence)
...

### 🔵 SUGGESTION — [Category]
**`path/to/file.ext:8`**  •  detected-by: Claude reasoning (low confidence)
...

## Convention violations
List any findings tied to a specific project doc with a link.

## Hotspot notes
Brief git-intelligence callouts for changed files (if any).

## Tooling suggestions
Tools that weren't installed but would have added depth. One-line each, optional.

## What looks good
2–4 bullets of positive patterns worth preserving.

## Recommended next steps
Ordered action list, most critical first.
```

### Severity guide

| Level | Use for |
|---|---|
| 🔴 Critical | Security vulnerabilities, credential leaks, known CVEs ≥7.0, crash bugs, data loss, secrets in git history |
| 🟡 Warning | Best-practice violations confirmed by tools, performance issues, hotspot files, moderate CVEs |
| 🔵 Suggestion | Style, minor improvements, low-confidence signals, tool installation hints |

---

## Tips

- **Cite `file:line`** on every finding. Without it, findings aren't actionable.
- **Cite tool provenance**: include `detected-by` so users know which tool flagged what.
- **Be opportunistic, not demanding** — never fail if a tool is missing; fall back to reasoning and suggest the install.
- **Do not repeat Claude's baseline** — the reference files are the curated list. Don't invent generic "best practice" findings that aren't in the refs. If a user wants that, they can ask Claude directly.
- **Respect the baseline** — a clean report against a baseline is success, not failure. Users adopt this skill on mature codebases; reporting legacy issues as if they were new creates noise.
- **Network calls permitted** for: OSV.dev (CVE scanning), package registries (license lookup), and tool auto-update. Not permitted: remote git hosts, PR systems, unrelated APIs.
