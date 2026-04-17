# Shell / Bash — Rules Reference

| Issue | Violation | Fix |
|---|---|---|
| Command injection via `eval` | `` eval `cmd $input` `` | Avoid `eval`; use arrays + `"${arr[@]}"` |
| Insecure temp file | `tmp=/tmp/myfile` (predictable, race-prone) | `tmp=$(mktemp)` |
| `curl \| sh` pattern | `curl https://... \| bash` | Download, verify checksum, then execute |
| Missing `--` before variable filenames | `rm $file` (fails if `$file` starts with `-`) | `rm -- "$file"` |

**Pruned** (handled by `shellcheck`): unquoted variables, missing `set -euo pipefail`, backticks, `IFS` issues, `[` vs `[[`.
