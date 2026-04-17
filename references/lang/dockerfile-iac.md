# Dockerfile / YAML / Terraform — Rules Reference

| Issue | Violation | Fix |
|---|---|---|
| Secrets baked into image | `ENV SECRET=abc123` | Build args + secrets mount; never bake |
| `latest` tag | `FROM node:latest` | Pin to digest or explicit version |
| Running as root | No `USER` directive | `USER node` (or any non-root) |
| Overly permissive IAM | `"Action": "*"`, `"Resource": "*"` | Least-privilege policy |
| `privileged: true` in pod spec | K8s/Docker privileged container | Remove unless absolutely required |
| Hardcoded credentials in YAML | `password: hunter2` | K8s Secrets / Vault references |

**Pruned** (handled by `hadolint` / `tfsec` / `checkov` / `kubesec`): apt-get caching, layer ordering, missing healthchecks, generic lints.
