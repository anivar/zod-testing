# Zod Testing Anti-Patterns

## Table of Contents

- Testing schema internals instead of behavior
- Not testing error paths
- Using parse() instead of safeParse() in tests
- Not testing boundary values
- Hardcoding mock data instead of generating
- Duplicating schema logic in assertions
- Testing transform output without testing input validation
- Not testing optional/nullable combinations
- Snapshot testing raw ZodError
- Not seeding random data generators

## Testing schema internals instead of behavior

```typescript
// BAD: testing internal structure — breaks on refactors
it("has correct shape", () => {
  expect(UserSchema.shape.name).toBeDefined()
  expect(UserSchema.shape.email).toBeDefined()
  expect(UserSchema._def.typeName).toBe("ZodObject")
})

// GOOD: test parse behavior — stable across refactors
it("accepts valid user", () => {
  const result = UserSchema.safeParse({
    name: "Alice",
    email: "alice@example.com",
  })
  expect(result.success).toBe(true)
})

it("rejects missing name", () => {
  const result = UserSchema.safeParse({ email: "alice@example.com" })
  expect(result.success).toBe(false)
})
```

## Not testing error paths

```typescript
// BAD: only testing valid inputs
describe("UserSchema", () => {
  it("accepts valid user", () => {
    expect(UserSchema.safeParse(validUser).success).toBe(true)
  })
  // No tests for invalid data — regressions go unnoticed
})

// GOOD: test both valid and invalid
describe("UserSchema", () => {
  it("accepts valid user", () => {
    expect(UserSchema.safeParse(validUser).success).toBe(true)
  })

  it("rejects empty name", () => {
    expect(
      UserSchema.safeParse({ ...validUser, name: "" }).success
    ).toBe(false)
  })

  it("rejects invalid email", () => {
    expect(
      UserSchema.safeParse({ ...validUser, email: "not-email" }).success
    ).toBe(false)
  })

  it("rejects negative age", () => {
    expect(
      UserSchema.safeParse({ ...validUser, age: -1 }).success
    ).toBe(false)
  })
})
```

## Using parse() instead of safeParse() in tests

```typescript
// BAD: test crashes with ZodError stack trace instead of clean failure
it("rejects invalid data", () => {
  expect(() => UserSchema.parse(invalidData)).toThrow()
  // If schema changes to accept this data, test still "passes"
  // because no throw means no assertion runs
})

// BAD: try/catch obscures the test intent
it("rejects invalid data", () => {
  try {
    UserSchema.parse(invalidData)
    fail("should have thrown")
  } catch (e) {
    expect(e).toBeInstanceOf(z.ZodError)
  }
})

// GOOD: clean, predictable assertions
it("rejects invalid data", () => {
  const result = UserSchema.safeParse(invalidData)
  expect(result.success).toBe(false)
  if (!result.success) {
    expect(result.error.issues).toHaveLength(2)
  }
})
```

## Not testing boundary values

```typescript
const AgeSchema = z.number().min(0).max(150)

// BAD: only testing middle values
it("validates age", () => {
  expect(AgeSchema.safeParse(25).success).toBe(true)
  expect(AgeSchema.safeParse(-5).success).toBe(false)
})

// GOOD: testing boundaries
it.each([
  [0, true],    // minimum boundary
  [150, true],  // maximum boundary
  [-1, false],  // one below minimum
  [151, false], // one above maximum
  [75, true],   // middle value
])("age %i → %s", (age, expected) => {
  expect(AgeSchema.safeParse(age).success).toBe(expected)
})
```

## Hardcoding mock data instead of generating

```typescript
// BAD: manual mock data drifts from schema
const mockUser = {
  name: "Test User",
  email: "test@test.com",
  age: 25,
  // forgot to add 'role' when schema was updated
}

it("accepts mock user", () => {
  expect(UserSchema.safeParse(mockUser).success).toBe(true) // fails silently
})

// GOOD: generate from schema — always in sync
import { fake, seed, install } from "zod-schema-faker"

beforeAll(() => install(z))
beforeEach(() => seed(42))

it("accepts generated user", () => {
  const user = fake(UserSchema)
  expect(UserSchema.safeParse(user).success).toBe(true)
})
```

## Duplicating schema logic in assertions

```typescript
// BAD: re-implementing validation in the test
it("validates email format", () => {
  const email = "test@example.com"
  const result = EmailSchema.safeParse(email)
  expect(result.success).toBe(true)
  // Duplicating the schema's own regex check in the test
  expect(email).toMatch(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)
})

// GOOD: trust the schema — test behavior, not implementation
it("validates email format", () => {
  expect(EmailSchema.safeParse("test@example.com").success).toBe(true)
  expect(EmailSchema.safeParse("not-an-email").success).toBe(false)
})
```

## Testing transform output without testing input validation

```typescript
const DateString = z.string()
  .refine((s) => !isNaN(Date.parse(s)), { error: "Invalid date" })
  .transform((s) => new Date(s))

// BAD: only testing the transform output
it("transforms to Date", () => {
  const result = DateString.parse("2024-01-01")
  expect(result).toBeInstanceOf(Date)
})

// GOOD: test both validation and transform
it("transforms valid date string", () => {
  const result = DateString.safeParse("2024-01-01")
  expect(result.success).toBe(true)
  if (result.success) {
    expect(result.data).toBeInstanceOf(Date)
    expect(result.data.getFullYear()).toBe(2024)
  }
})

it("rejects invalid date string", () => {
  const result = DateString.safeParse("not-a-date")
  expect(result.success).toBe(false)
})
```

## Not testing optional/nullable combinations

```typescript
const Schema = z.object({
  name: z.string(),
  bio: z.string().optional(),
  avatar: z.string().nullable(),
})

// BAD: only testing with all fields present
it("accepts user", () => {
  expect(Schema.safeParse({
    name: "Alice",
    bio: "Hello",
    avatar: "https://example.com/img.png",
  }).success).toBe(true)
})

// GOOD: test all optional/nullable variants
it("accepts without optional field", () => {
  expect(Schema.safeParse({ name: "Alice", avatar: null }).success).toBe(true)
})

it("accepts with null nullable", () => {
  expect(Schema.safeParse({ name: "Alice", avatar: null }).success).toBe(true)
})

it("rejects null for non-nullable optional", () => {
  expect(Schema.safeParse({ name: "Alice", bio: null, avatar: null }).success).toBe(false)
})

it("rejects undefined for non-optional nullable", () => {
  expect(Schema.safeParse({ name: "Alice" }).success).toBe(false)
  // avatar is nullable but not optional — must be explicitly provided
})
```

## Snapshot testing raw ZodError

```typescript
// BAD: raw ZodError snapshots are noisy and brittle
it("error matches snapshot", () => {
  const result = schema.safeParse(invalid)
  if (!result.success) {
    expect(result.error).toMatchSnapshot() // huge, includes internals
  }
})

// GOOD: snapshot formatted output
it("error matches snapshot", () => {
  const result = schema.safeParse(invalid)
  if (!result.success) {
    expect(z.flattenError(result.error)).toMatchSnapshot()
    // Clean: { formErrors: [], fieldErrors: { email: ["Invalid email"] } }
  }
})

// GOOD: snapshot JSON Schema for schema shape
it("schema shape matches snapshot", () => {
  expect(z.toJSONSchema(schema)).toMatchSnapshot()
})
```

## Not seeding random data generators

```typescript
// BAD: non-deterministic — test passes sometimes, fails others
it("accepts generated data", () => {
  const user = fake(UserSchema) // different every run
  expect(UserSchema.safeParse(user).success).toBe(true)
  // If this fails, you can't reproduce it
})

// GOOD: seeded — deterministic and reproducible
beforeEach(() => seed(42))

it("accepts generated data", () => {
  const user = fake(UserSchema) // same every run
  expect(UserSchema.safeParse(user).success).toBe(true)
})
```
