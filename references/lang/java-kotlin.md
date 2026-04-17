# Java / Kotlin — Rules Reference

| Issue | Violation | Fix |
|---|---|---|
| Java: `equals`/`hashCode` contract broken | Overriding one without the other | Always override both (`Objects.hash(...)`) |
| Java: `Collections.unmodifiableList` still mutable at source | `return Collections.unmodifiableList(srcHeld)` | `return List.copyOf(src)` (Java 10+) |
| Kotlin: `GlobalScope.launch` leak | `GlobalScope.launch { ... }` without cancellation | `viewModelScope.launch` / `lifecycleScope.launch` |
| Kotlin: cold `Flow` for UI state | `Flow<List<T>>` resubscribes on recomposition | `.stateIn(scope, SharingStarted.WhileSubscribed(), default)` |
| Kotlin: `runBlocking` on main thread | `runBlocking { fetchData() }` on Android main | `lifecycleScope.launch(Dispatchers.IO) { fetchData() }` |

**Pruned** (handled by `spotbugs` / `detekt` / `ktlint`): swallowed exceptions, `!!` operator, missing `data class`, NPE risks.
