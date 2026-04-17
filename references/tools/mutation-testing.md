# Mutation Testing (Opt-in)

Coverage percentage is a weak signal. Mutation testing actually measures whether tests **catch bugs** by mutating source code and checking whether tests fail.

## Tools

| Language | Tool | Command |
|---|---|---|
| Python | `mutmut` | `mutmut run` |
| JS/TS | `stryker` | `npx stryker run` |
| Rust | `cargo-mutants` | `cargo mutants` |
| Ruby | `mutant` | `mutant run` |
| Java | `pitest` | `mvn org.pitest:pitest-maven:mutationCoverage` |

## When to Run

**Only on demand** — mutation testing is slow (minutes to hours). Suggest it in the report when:
- Coverage is high (>80%) but the user asked for a deep review
- User explicitly opts in

Do NOT run by default. It burns budget and time disproportionate to its value for most reviews.

## Output Interpretation

Mutation score = (killed mutants) / (total mutants). Higher is better.
- >80%: excellent
- 60-80%: good
- <60%: tests exist but don't exercise critical paths
