# Rust — Rules Reference

| Issue | Violation | Fix |
|---|---|---|
| `unwrap()` / `expect()` in non-test paths | `value.unwrap()` | Propagate with `?` or handle explicitly |
| `unsafe` block without `// SAFETY:` comment | `unsafe { ... }` | Document invariants being upheld |
| Integer overflow in release mode | Unchecked `+`/`*` on `usize`/`u32` | `checked_add`, `saturating_add`, or `wrapping_add` |
| Unhandled `Mutex` poison | `lock.unwrap()` after potential panic | Match on `PoisonError`, decide recovery |

**Pruned** (handled by `cargo clippy` / `cargo-udeps`): `clone()` on large types, missing `?` propagation, `todo!()` in production, `std::mem::forget` patterns, generic style lints.
