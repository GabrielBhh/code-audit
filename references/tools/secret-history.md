# Secrets in Git History

Load in Step 5 (secret history scanning). Current-file scanning misses secrets committed and later deleted — git history retains them.

## Tools

| Tool | Command | Notes |
|---|---|---|
| `gitleaks` | `gitleaks detect --source . --report-format json --no-banner` | Fast, actively maintained |
| `trufflehog` | `trufflehog git file://. --json` | Verifies secrets against live services |
| `git-secrets` | `git-secrets --scan-history` | AWS-focused, older |

## Severity

**Always 🔴 Critical** when a secret is detected in history — even if deleted from HEAD:
- The commit SHA is public the moment it was pushed
- Rotating the credential is the only remediation
- `git filter-repo` cleanup is required to remove it from history

## Fallback

If no scanner is installed, emit a single 🔵 Suggestion: "install `gitleaks` (`brew install gitleaks`)" — don't scan history manually (too slow, too noisy).
