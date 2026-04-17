# Go — Rules Reference

## Correctness & Security

| Issue | Violation | Fix |
|---|---|---|
| `fmt.Sprintf` in SQL | `fmt.Sprintf("SELECT ... %s", val)` | `db.Query("SELECT ... $1", val)` |
| Goroutine closure capture in loop | `for _, i := range xs { go func() { use(i) }() }` | `go func(i int) { use(i) }(i)` |
| Concurrent map write | Unsynchronized writes from goroutines | `sync.RWMutex` or `sync.Map` |
| `defer` inside loop | `for ... { defer f.Close() }` | Move close outside or extract helper |
| Goroutine leak | Goroutine with no cancel/stop mechanism | `context.Context` with cancellation |

## Simplification

| Issue | Violation | Fix |
|---|---|---|
| String comparison for error type | `if err.Error() == "not found"` | `if errors.Is(err, ErrNotFound)` |
| Unwrapped error | `return fmt.Errorf("failed: %s", err)` | `return fmt.Errorf("failed: %w", err)` |
| String concat in loop | `s += part` in loop | `var sb strings.Builder; sb.WriteString(part)` |
| Repeated test structure | Near-identical test functions | Table-driven: `tests := []struct{input, want}{...}` |
| Named return used inconsistently | Named returns declared but not used everywhere | Use consistently or remove |
| `interface{}` / `any` overuse | `func f(v interface{})` for a known type | Concrete type or constrained generic |
| Unnecessary pointer to small value | `*bool`, `*int` for optional | Wrapper struct or zero-value sentinel |

**Pruned** (handled by `go vet` / `staticcheck` / `golangci-lint`): ignored error return, `panic` vs error, naked returns in long funcs.
