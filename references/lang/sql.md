# SQL — Rules Reference

## Correctness & Security

| Issue | Violation | Fix |
|---|---|---|
| String interpolation | `"SELECT ... WHERE id = " + id` | Parameterized/prepared statements |
| Missing index on FK | FK column with no index | `CREATE INDEX idx_orders_user_id ON orders(user_id)` |
| Leading wildcard defeats index | `WHERE name LIKE '%smith'` | Full-text search or reversed-string suffix index |
| `NOT IN` with nullable subquery | `WHERE id NOT IN (SELECT user_id FROM orders)` when `user_id` is nullable | `WHERE NOT EXISTS (SELECT 1 FROM orders ...)` |
| Multi-step write without transaction | Sequential `INSERT`/`UPDATE` with no `BEGIN`/`COMMIT` | Wrap in a transaction |
| Composite index column order | Index `(status, id)` used in query filtering only on `id` | Put most-queried / most-selective column first |

## Simplification

| Issue | Violation | Fix |
|---|---|---|
| Subquery where `JOIN` suffices | `SELECT * FROM orders WHERE user_id IN (SELECT id FROM users ...)` | Rewrite as `JOIN` |
| `HAVING` without `GROUP BY` | `HAVING COUNT(*) > 1` with no `GROUP BY` | Add `GROUP BY` or use `WHERE` |
| Redundant `DISTINCT` | `SELECT DISTINCT` when result is already unique | Remove — may mask missing `JOIN` conditions |
| `OR` on indexed column | `WHERE status = 'a' OR status = 'b'` | `WHERE status IN ('a', 'b')` |
| Correlated subquery in `SELECT` | `SELECT (SELECT name FROM ...) FROM ...` per-row | Rewrite as `LEFT JOIN` |

**Pruned** (generic linting): `SELECT *`, missing `LIMIT`, implicit type coercion.
