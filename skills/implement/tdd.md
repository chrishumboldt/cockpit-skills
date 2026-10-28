# Test-first: the loop and what makes a good test

Reference for the Implement step's red → green loop. Read `GLOSSARY.md` first, so test names use the project's terms.

## The loop

- **Red before green.** Write one failing test, watch it fail for the reason you expect, then write only enough code to pass it.
- **One slice at a time.** One seam, one test, one minimal change per cycle. Each test is a **tracer bullet**: it answers what the last cycle taught you.
- **Vertical, never horizontal.** Writing all the tests first and then all the code tests the shape you imagined, not the behaviour you built.
- **Refactor on green.** Tidy only with every test passing, and change no behaviour while you do.

## Seams

A **seam** is the public boundary you test at: the interface where behaviour shows without reaching inside. Test at the seams the spec names. When the ticket needs one the spec doesn't name, take the highest one the code offers and say so in `build.md`.

## A good test

Tests check behaviour through public interfaces, so the code behind them can change entirely and they still pass. A good test reads like a line of the spec: "a failed step pauses the run" names a capability and survives a refactor.

```typescript
// Good: observable behaviour, through the interface
test('a created user can be read back', async () => {
  const user = await createUser({ name: 'Alice' })
  expect((await getUser(user.id)).name).toBe('Alice')
})

// Bad: goes round the interface to check a side effect
test('createUser saves to the database', async () => {
  await createUser({ name: 'Alice' })
  expect(await db.query('SELECT * FROM users WHERE name = ?', ['Alice'])).toBeDefined()
})
```

Signs a test is coupled to the implementation: it mocks your own modules, tests a private function, asserts on call counts or order, or breaks on a refactor that changed no behaviour.

**Expected values come from outside the code.** An assertion that recomputes the answer the way the code does passes by construction:

```typescript
// Bad: restates the implementation
expect(total(items)).toBe(items.reduce((sum, i) => sum + i.price, 0))
// Good: a known literal
expect(total([{ price: 10 }, { price: 5 }])).toBe(15)
```

## Mocks

Mock at **system boundaries** only: external APIs, time and randomness, and sometimes the database or the file system (a real test database or a temp folder is better). Never mock your own modules or anything else you control.

Make boundaries easy to replace: pass the dependency in rather than building it inside, and give an external service one function per operation (`getUser`, `createOrder`) rather than one generic `fetch`, so each fake returns one shape.
