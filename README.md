# Zod Testing

An AI agent skill for testing Zod schemas with Jest and Vitest.

## The Problem

AI agents often write schema tests that only check the happy path, use `parse()` instead of `safeParse()` (crashing instead of failing), test schema internals (`.shape`, `._def`) instead of behavior, or hardcode mock data instead of generating it from the schema. The result: tests that miss regressions and break on harmless refactors.

## This Solution

A focused testing skill covering schema correctness testing, error assertion patterns, mock data generation, snapshot testing with `z.toJSONSchema()`, property-based testing, structural testing, and drift detection — with 14 anti-patterns showing exactly what goes wrong and how to fix it.

## Install

```bash
npx skills add anivar/zod-testing -g
```

Or with full URL:

```bash
npx skills add https://github.com/anivar/zod-testing
```

## Baseline

- zod ^4.0.0
- Jest or Vitest
- TypeScript ^5.5

## What's Inside

### Testing Approaches

| Approach | Type | Use When |
|----------|------|----------|
| `safeParse()` result checking | Correctness | Default — always use safeParse in tests |
| `z.flattenError()` assertions | Error messages | Verifying specific field errors |
| `z.toJSONSchema()` snapshots | Schema shape | Detecting unintended schema changes |
| Mock data generation | Fixtures | Need valid/randomized test data |
| Property-based testing | Fuzz testing | Schemas must handle arbitrary valid inputs |
| Structural testing | Architecture | Verify schemas are only imported at boundaries |
| Drift detection | Regression | Catch unintended schema changes via JSON Schema snapshots |

### Anti-Patterns

14 common testing mistakes with BAD/GOOD code examples:
- Testing schema internals instead of behavior
- Not testing error paths (only happy path)
- Using `parse()` in tests (crashes instead of failing)
- Not testing boundary values (min/max edges)
- Hardcoding mock data instead of generating from schema
- Snapshot testing raw ZodError instead of formatted output
- Not seeding random data generators (flaky tests)
- Not testing at boundaries (schema tests pass but handler doesn't validate)
- No snapshot regression testing (field removal goes unnoticed)
- Testing schema shape but not error observability
- No drift detection workflow (schema changes land without review)

## Structure

```
├── SKILL.md                      # Entry point for AI agents
└── references/
    ├── api-reference.md          # Testing patterns, assertion helpers, mock generation
    └── anti-patterns.md          # Common testing mistakes to avoid
```

## Related

- [zod-skill](https://github.com/anivar/zod-skill) — Full Zod v4 best practices skill (27 rules across parsing, schema design, error handling, architecture, observability, and more)

## License

MIT
