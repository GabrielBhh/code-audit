# Ruby — Rules Reference

## Correctness & Security

| Issue | Violation | Fix |
|---|---|---|
| Mass assignment (Rails) | `User.new(params)` | Strong params: `params.require(:user).permit(:name, :email)` |
| SQL injection via string | `User.where("name = '#{name}'")` | `User.where(name: name)` |
| Missing `dependent:` on `has_many` | `has_many :posts` | `has_many :posts, dependent: :destroy` |
| `Time.now` ignores Rails timezone | `Time.now` in a Rails app | `Time.current` or `Time.zone.now` |
| `update_attribute` skips validations | `user.update_attribute(:role, :admin)` | `user.update(role: :admin)` — or document bypass |

## Simplification

| Issue | Violation | Fix |
|---|---|---|
| `each` + `push` instead of `map` | `result = []; items.each { \|i\| result << f(i) }` | `result = items.map { \|i\| f(i) }` |
| `select` + `first` instead of `find` | `items.select { \|i\| pred(i) }.first` | `items.find { \|i\| pred(i) }` |
| `map` + `compact` instead of `filter_map` | `items.map { \|i\| maybe(i) }.compact` | `items.filter_map { \|i\| maybe(i) }` (Ruby 2.7+) |
| Manual presence check | `if str && !str.empty?` | `if str.present?` (Rails) |
| Interpolation in single-quoted string | `'Hello #{name}'` (won't interpolate) | `"Hello #{name}"` |

**Pruned** (handled by `rubocop` / `brakeman`): N+1 detection, `rescue` without type, missing `frozen_string_literal`, `Symbol#to_proc`, `||=` on falsy.
