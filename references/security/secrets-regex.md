# Secret Detection Patterns

Load in the secret-scanning step (Step 5 of the pipeline). Any match is 🔴 Critical.

| Secret type | Regex pattern |
|---|---|
| AWS Access Key | `AKIA[0-9A-Z]{16}` |
| AWS Secret Key | `(?i)aws.{0,20}secret.{0,20}['\"][0-9a-zA-Z/+]{40}` |
| GitHub Token | `ghp_[a-zA-Z0-9]{36}` or `github_pat_[a-zA-Z0-9_]{82}` |
| Stripe Live Key | `sk_live_[0-9a-zA-Z]{24,}` |
| Stripe Test Key | `sk_test_[0-9a-zA-Z]{24,}` |
| SendGrid API Key | `SG\.[a-zA-Z0-9_-]{22,}\.[a-zA-Z0-9_-]{43,}` |
| Slack Token | `xox[baprs]-[0-9a-zA-Z-]{10,}` |
| Private Key Block | `-----BEGIN (RSA\|EC\|OPENSSH\|PGP) PRIVATE KEY` |
| Generic Password | `(?i)(password\|passwd\|pwd)\s*=\s*['"][^'"]{4,}['"]` |
| DB Connection String | `(?i)(postgres\|mysql\|mongodb):\/\/[^:]+:[^@]+@` |
| JWT | `eyJ[a-zA-Z0-9_-]+\.eyJ[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+` |
| Anthropic API Key | `sk-ant-api03-[a-zA-Z0-9\-_]{93}` |
| OpenAI API Key | `sk-[a-zA-Z0-9]{48}` |
| Google API Key | `AIza[0-9A-Za-z\-_]{35}` |
| Twilio Auth Token | `(?i)twilio.{0,20}['\"][0-9a-f]{32}['\"]` |
| Env-var fallback (hardcoded) | `os\.getenv\(['\"][^'\"]+['\"],\s*['\"][^'\"]{4,}['\"]` — second arg is a hardcoded default |

**Fix for all secrets**: replace with `os.environ["VAR_NAME"]` (or the language's equivalent) and **rotate the credential immediately** — the value is in git history even if deleted from the current file.
