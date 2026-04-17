# C / C++ — Rules Reference

| Issue | Violation | Fix |
|---|---|---|
| Unbounded string copy | `strcpy(buf, src)` / `gets(buf)` | `strncpy(buf, src, sizeof(buf)-1)` / `fgets` |
| Format string injection | `printf(user_input)` | `printf("%s", user_input)` |
| Use-after-free | Pointer used after `delete` / `free` | Smart pointers; null out after free |
| Double-free | `delete ptr` called twice | `unique_ptr` / set to `nullptr` after free |
| `sprintf` without bounds | `sprintf(buf, fmt, val)` | `snprintf(buf, sizeof(buf), fmt, val)` |
| Signed integer overflow (UB) | `INT_MAX + 1` | `__builtin_add_overflow` or `<limits>` guards |

**Pruned** (handled by `clang-tidy` / `cppcheck` / `scan-build`): memory leaks, null pointer dereference, missing `const`, raw owning pointers in modern C++.
