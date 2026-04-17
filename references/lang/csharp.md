# C# / .NET — Rules Reference

| Issue | Violation | Fix |
|---|---|---|
| `async void` method (non-event handler) | `async void HandleClick(...)` | `async Task HandleClick(...)` — `async void` swallows exceptions |
| Missing `ConfigureAwait(false)` in library code | `await SomeAsync()` | `await SomeAsync().ConfigureAwait(false)` |
| `First()` NRE risk | `list.First(x => x.Id == id)` | `list.FirstOrDefault(...)` with null check |
| `DateTime.Now` as a timestamp | `DateTime.Now` written to DB | `DateTime.UtcNow` |
| `Thread.Sleep` in async context | `Thread.Sleep(1000)` inside `async` method | `await Task.Delay(1000)` |

**Pruned** (handled by Roslyn analyzers): `IDisposable` not in `using`, string concat in loops, hardcoded connection strings, SQL interpolation.
