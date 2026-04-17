# Python — Rules Reference

## Correctness & Security

| Issue | Violation | Fix |
|---|---|---|
| Mutable default argument | `def f(items=[])` | `def f(items=None): ...` |
| `yaml.load()` without `Loader=` | `yaml.load(f)` | `yaml.safe_load(f)` |
| `pickle.loads` on untrusted data | `pickle.loads(user_data)` | JSON or signed/validated serializer |
| Hardcoded env var fallback | `os.getenv("SECRET_KEY", "dev-secret")` | `os.environ["SECRET_KEY"]` — fail loudly |
| Non-constant-time secret comparison | `if token == expected:` | `hmac.compare_digest(token, expected)` |
| File read before size check | `data = await file.read(); if len(data) > MAX:` | Check `Content-Length` first, or `file.read(MAX + 1)` |
| Unconstrained enum-like field (Pydantic) | `status: str` | `status: Literal["active", "inactive"]` or `Enum` |
| Privileged route missing role dep (FastAPI) | `@router.put("/settings/api-keys")` with only auth | `current_user: User = Depends(require_admin)` |
| Known unmaintained package | `python-jose`, `pycrypto` imported | `authlib`/`PyJWT`; `pycrypto` → `cryptography` |
| SSTI via `render_template_string` | `render_template_string(user_input)` | `render_template("file.html", value=user_input)` |
| JWT algorithm not pinned | `jwt.decode(token, key)` | `jwt.decode(token, key, algorithms=["HS256"])` |

## Simplification

| Issue | Violation | Fix |
|---|---|---|
| `len()` check for emptiness | `if len(items) == 0:` | `if not items:` |
| `range(len())` loop | `for i in range(len(items)):` | `for item in items:` / `enumerate(items)` |
| `lambda` in `map`/`filter` | `map(lambda x: x.name, items)` | `[item.name for item in items]` |
| Loop building a list | `result = []; for x in xs: result.append(f(x))` | `[f(x) for x in xs]` |
| Loop building a dict | `d = {}; for k, v in pairs: d[k] = v` | `{k: v for k, v in pairs}` |
| Manual `any`/`all` loop | Manual iteration returning bool | `any(pred(x) for x in xs)` |
| Old-style string formatting | `"Hello %s" % name` / `"{}".format(name)` | `f"Hello {name}"` |
| `os.path` instead of `pathlib` | `os.path.join(base, "sub", "file.txt")` | `Path(base) / "sub" / "file.txt"` |
| `dict()` / `list()` constructors | `d = dict()` | `d = {}` |
| Manual parallel index loop | `for i in range(len(a)): use(a[i], b[i])` | `for x, y in zip(a, b):` |
| `try/except/else` missing `else` | Logic after `try` inside `try` | Move non-raising code to `else` |
| `dict.get()` opportunity | `val = d[k] if k in d else default` | `val = d.get(k, default)` |
| `defaultdict` opportunity | Repeated `if key not in d: d[key] = []` | `defaultdict(list)` |
| Unpacking opportunity | `x = pair[0]; y = pair[1]` | `x, y = pair` |

**Pruned from this file** (handled by linters — `ruff`, `mypy`, `bandit`): bare `except:`, missing type hints, `print()` in production, wildcard imports, global mutable state.
