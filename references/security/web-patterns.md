# Web Security Patterns

Load for web-facing code: HTTP handlers, templating, client-side JS, API endpoints.

## SQL Injection

```python
# BAD
query = f"SELECT * FROM users WHERE email = '{email}'"

# GOOD
cursor.execute("SELECT * FROM users WHERE email = %s", (email,))
```

## Shell Injection

```python
# BAD
os.system(f"convert {filename} output.png")
subprocess.run(f"grep {search_term} logs.txt", shell=True)

# GOOD
subprocess.run(["convert", filename, "output.png"])
```

## XSS

```javascript
// BAD
element.innerHTML = userInput
document.write(userInput)

// GOOD
element.textContent = userInput
// or: DOMPurify.sanitize(userInput) if HTML is required
```

## Server-Side Template Injection (SSTI)

```python
# BAD — Jinja2 renders user input as a template
env = Environment()
template = env.from_string(user_input)

# GOOD
template = env.get_template("fixed_template.html")
template.render(user_value=user_input)
```

## XML External Entity (XXE)

```python
# BAD — lxml resolves external entities by default
tree = etree.parse(user_file)

# GOOD
parser = etree.XMLParser(resolve_entities=False, no_network=True)
tree = etree.parse(user_file, parser)
```

## ReDoS — Catastrophic Backtracking

```python
# BAD — nested quantifiers on overlapping classes
re.match(r"(a+)+$", user_input)
re.match(r"(\w+\s?)+$", user_input)

# GOOD — possessive quantifiers, atomic groups, or validate input length
```

## CSRF & Security Headers

- State-changing endpoints (POST/PUT/DELETE) on cookie-authenticated apps must verify a CSRF token or use `SameSite=Strict`
- Response headers that should be present:
  - `Strict-Transport-Security: max-age=63072000; includeSubDomains`
  - `Content-Security-Policy: default-src 'self'`
  - `X-Frame-Options: DENY` or `SAMEORIGIN`
  - `X-Content-Type-Options: nosniff`
  - `Referrer-Policy: strict-origin-when-cross-origin`

## CORS

- `Access-Control-Allow-Origin: *` on authenticated endpoints = broken access control

## Open Redirect

```python
# BAD
return redirect(request.args.get("next"))

# GOOD
next_url = request.args.get("next", "/")
if next_url not in ALLOWED_REDIRECT_DOMAINS:
    next_url = "/"
return redirect(next_url)
```

## PHP-specific patterns

```php
// BAD — SQL injection
$result = mysqli_query($conn, "SELECT * FROM users WHERE id = " . $_GET['id']);

// GOOD — prepared statement
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$_GET['id']]);

// BAD — type juggling allows "0abc" == 0 → true
if ($_POST['role'] == 0) { grant_admin(); }

// GOOD
if ($_POST['role'] === 0) { grant_admin(); }
```
