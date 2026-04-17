# Native Code Security Patterns

Load for C, C++, and (selectively) Rust `unsafe` blocks.

## Memory Safety

- **Buffer overflow** — `strcpy(buf, src)`, `gets(buf)`, `sprintf(buf, fmt, val)` with unbounded inputs → use `strncpy`, `fgets`, `snprintf`
- **Use-after-free** — pointer dereferenced after `free`/`delete` → prefer `std::unique_ptr` (C++) or null out after free
- **Double-free** — `free(p)` / `delete p` called twice → smart pointers or null-after-free pattern
- **Null pointer dereference** — unchecked pointer used in arithmetic or deref → check before use
- **Format string injection** — `printf(user_input)` allows `%n`/`%x` attacks → `printf("%s", user_input)`

## Undefined Behaviour

- **Signed integer overflow** — `INT_MAX + 1` is UB in C → `__builtin_add_overflow` or `<limits>` guards
- Reading uninitialised memory
- Strict-aliasing violations (type-punning via casts)

## Rust-Specific

- `unsafe` block without a `// SAFETY:` comment → must document invariants being upheld
- `std::mem::forget` / manual `ManuallyDrop` without justification
- `unwrap()` in `unsafe` contexts — if the unwrap panics, the safety reasoning may also be compromised

## Concurrency (C/C++)

- Data race: shared variable accessed from multiple threads without synchronization
- Lock ordering inconsistency → deadlock risk
- Incorrect `volatile` vs atomic usage
