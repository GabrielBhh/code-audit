# Dart / Flutter — Rules Reference

| Issue | Violation | Fix |
|---|---|---|
| `setState` after `dispose` (crash) | `setState(() { ... })` in async callback | `if (mounted) setState(...)` |
| Async work in `build()` | `Future.delayed(...)` inside `build` | Move to `initState` / controller |
| `BuildContext` across async gap | `context.push(...)` after `await` without check | `if (context.mounted) context.push(...)` |
| `initState` calling `async` directly without catch | `initState() { fetchData(); }` | `.then((v) { if (mounted) setState(...); }).catchError(...)` |

**Pruned** (handled by Dart analyzer): missing `const`, uncaught `Future`, deep widget nesting, `print()` in production.
