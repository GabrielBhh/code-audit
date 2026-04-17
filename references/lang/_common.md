# Language-Agnostic Simplification Patterns

Apply these to any language. Flag as 🔵 Suggestion unless the pattern masks a bug (then 🟡 Warning).

| Issue | Violation | Fix |
|---|---|---|
| Redundant boolean return | `if cond: return True` / `else: return False` | `return cond` |
| Redundant boolean comparison | `if x == True:` / `if x == False:` | `if x:` / `if not x:` |
| Unnecessary variable before return | `result = compute(); return result` | `return compute()` |
| Nested `if` instead of guard clause | Deep nesting for happy path | Invert condition and return/raise early |
| Unnecessary `else` after `return`/`raise` | `if cond: return x; else: return y` | `if cond: return x` / `return y` |
| Magic number / magic string | `if status == 3:` / `time.sleep(86400)` | Named constant: `PENDING = 3` / `SECONDS_PER_DAY = 86400` |
| Too many function parameters | `def f(a, b, c, d, e, f)` (>4 params) | Group into a config dataclass/struct/object |
| Function violating SRP | One function fetches, transforms, validates, and saves | Split into focused single-purpose functions |
| Double negation | `if not not x:` / `if not (a != b):` | `if x:` / `if a == b:` |
| Comparing to `None`/`null` with `==` | `if x == None:` | Use identity check in the language's idiom (`is None`, `=== null`, `== nil`) |
