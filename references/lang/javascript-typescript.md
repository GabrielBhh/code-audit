# JavaScript / TypeScript — Rules Reference

## Correctness & Security

| Issue | Violation | Fix |
|---|---|---|
| `innerHTML` from user input | `el.innerHTML = input` | `el.textContent = input`, or `DOMPurify.sanitize(input)` |
| N+1 awaits in a loop | `for (u of users) await fetch(u.id)` | `await Promise.all(users.map(u => fetch(u.id)))` |
| `eval()` on external input | `eval(userInput)` | Never; refactor to parser/lookup |
| `dangerouslySetInnerHTML` unsanitized (React) | `<div dangerouslySetInnerHTML={{__html: content}}>` | Sanitize with DOMPurify or render as text |
| `useEffect` missing cleanup | `useEffect(() => { subscribe() }, [])` with no return | `return () => unsubscribe()` |
| Stale closure in `useEffect` / `useCallback` | deps array stale; referenced var doesn't re-trigger | Add to deps, or `useRef` / functional `setState` |

## Simplification

| Issue | Violation | Fix |
|---|---|---|
| Manual null/undefined chain | `a && a.b && a.b.c` | `a?.b?.c` |
| Manual nullish default | `val !== null && val !== undefined ? val : def` | `val ?? def` |
| `filter` + `map` two passes | `arr.filter(pred).map(fn)` | Single `reduce` or `flatMap` if heavy |
| Unnecessary array spread | `[...arr].map(fn)` (no mutation) | `arr.map(fn)` |
| `!!x` for boolean cast | `const flag = !!value` | `const flag = Boolean(value)` |
| String concatenation | `"Hello " + name + "!"` | `` `Hello ${name}!` `` |
| `Object.assign({}, obj)` shallow clone | `Object.assign({}, obj)` | `{ ...obj }` |
| Callback-style promise | `.then(res => { ... }).then(val => { ... })` | `async/await` |
| `indexOf !== -1` existence check | `arr.indexOf(x) !== -1` | `arr.includes(x)` |
| Manual object iteration | `Object.keys(obj).forEach(k => use(k, obj[k]))` | `Object.entries(obj).forEach(([k, v]) => use(k, v))` |
| Redundant `return` in arrow | `x => { return x * 2; }` | `x => x * 2` |
| TS: unnecessary type assertion | `x as string` when already `string` | Remove assertion |
| TS: single-use interface | Overly abstract interface used once | Inline the type or use `type` alias |

**Pruned** (handled by ESLint/biome/tsc): `var` → `const/let`, `== vs ===`, missing `await`, `any` type, `console.log` in production, missing React `key`, unhandled promise rejections.
