# PHP — Rules Reference

| Issue | Violation | Fix |
|---|---|---|
| SQL injection via concatenation | `"SELECT ... WHERE id = " . $_GET['id']` | PDO prepared statements |
| XSS via direct output | `echo $_GET['name']` | `echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8')` |
| `eval()` on user input | `eval($_POST['code'])` | Never |
| Weak password hashing | `md5($password)` / `sha1($password)` | `password_hash($password, PASSWORD_BCRYPT)` |
| SSRF via user-supplied URL | `file_get_contents($_GET['url'])` | Validate against allowlist |
| Type juggling with `==` | `if ($token == 0)` allows `"0abc" == 0` → true | `if ($token === 0)` |
| Variable injection | `extract($_REQUEST)` | Never on user input |
| Unvalidated file upload | `move_uploaded_file(...)` without checks | Validate MIME, extension, store outside webroot |

**Pruned** (handled by `phpstan` / `psalm`): generic type hints, unused variables, style cops.
