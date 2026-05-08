---
name: tdd
description: Test-driven development with red-green-refactor loop for Ruby on Rails. Use when building features or fixing bugs using TDD, mentions "red-green-refactor", wants integration tests, or asks for test-first development. Uses Minitest and fixtures.
---

# Test-Driven Development (Rails / Minitest)

## Philosophy

**Core principle**: Tests should verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't.

**Good tests** exercise real code paths through public APIs. They describe _what_ the system does, not _how_. A good test reads like a specification — "user can check out with a valid cart" tells you exactly what capability exists. These tests survive refactors because they don't care about internal structure.

**Bad tests** are coupled to implementation. They test private methods, stub internals, or verify through back-door database queries instead of going through the application's own interface. Warning sign: your test breaks on a refactor but behavior hasn't changed.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for stubbing guidelines.

## Test Types in Rails

Use the right scope for the job:

- **Unit tests** (`test/models/`, `test/lib/`) — pure Ruby objects, model validations, business logic with no HTTP
- **Controller tests** (`test/controllers/`) — request/response cycle, redirects, status codes; avoid asserting on view details
- **Integration tests** (`test/integration/`) — multi-step flows across controllers; preferred over controller tests for complex flows
- **System tests** (`test/system/`) — full browser via Capybara; use sparingly, only for critical user journeys

Default to the lowest scope that gives you confidence. System tests are slow; unit tests are fast. Most behavior belongs in unit or integration tests.

## Fixtures

Use Rails fixtures (`test/fixtures/*.yml`). They are fast, deterministic, and already in the database when your test runs.

**Good fixture practices:**
- Name fixtures after roles, not identifiers: `users.yml` has `admin:`, `member:`, `guest:` — not `user_1:`, `user_2:`
- Keep fixture files lean; only include attributes relevant to tests
- Use ERB for dynamic values: `created_at: <%= 3.days.ago %>`
- Reference associations by fixture name: `author: admin`

**Avoid FactoryBot.** Fixtures are Rails-native, commit-time validated, and eliminate factory explosion. If you find yourself needing highly varied object states, that is a sign the behavior under test is too broad — narrow the test first.

## Anti-Pattern: Horizontal Slices

**DO NOT write all tests first, then all implementation.** This produces tests that verify imagined behavior, not actual behavior.

**Correct approach**: One test → one implementation → repeat. Each test responds to what you learned from the previous cycle.

```
WRONG (horizontal):
  RED:   test1, test2, test3, test4, test5
  GREEN: impl1, impl2, impl3, impl4, impl5

RIGHT (vertical):
  RED→GREEN: test1→impl1
  RED→GREEN: test2→impl2
  ...
```

## Workflow

### 1. Planning

Before writing any code:

- [ ] Confirm which behaviors to test (prioritize critical paths)
- [ ] Identify the right test scope (unit / controller / integration / system)
- [ ] Check existing fixtures — use what's there before adding new ones
- [ ] Design the public interface first
- [ ] Get user approval on the plan

### 2. Tracer Bullet

Write ONE test that confirms ONE thing end-to-end:

```ruby
# test/models/order_test.rb
class OrderTest < ActiveSupport::TestCase
  test "calculates total from line items" do
    order = orders(:with_two_items)
    assert_equal 50_00, order.total_cents
  end
end
```

Run it: `bin/rails test test/models/order_test.rb`

Watch it fail (RED). Write the minimal code to pass (GREEN).

### 3. Incremental Loop

For each remaining behavior:

```
RED:   Write next test → bin/rails test → fails
GREEN: Minimal code to pass → bin/rails test → passes
```

Rules:
- One test at a time
- Only enough code to pass the current test
- Don't anticipate future tests

### 4. Refactor

After all tests pass, look for [refactor candidates](refactoring.md):

- [ ] Extract duplication
- [ ] Push logic into models (fat model, thin controller)
- [ ] Consider whether a service object or PORO improves clarity
- [ ] Run `bin/rails test` after each refactor step

**Never refactor while RED.** Get to GREEN first.

## Rails-Specific Patterns

### Testing ActiveRecord models

Test validations, scopes, and instance methods through the model interface — not by querying the database directly.

```ruby
test "requires email" do
  user = User.new(email: nil)
  assert_not user.valid?
  assert_includes user.errors[:email], "can't be blank"
end

test "active scope excludes suspended users" do
  assert_includes User.active, users(:active_member)
  assert_not_includes User.active, users(:suspended_member)
end
```

### Testing controllers

Use `assert_response`, `assert_redirected_to`, and `assert_difference` — not view content.

```ruby
test "creates order and redirects" do
  assert_difference "Order.count" do
    post orders_path, params: { order: { item_id: items(:widget).id } }
  end
  assert_redirected_to order_path(Order.last)
end
```

### Testing with authentication

Set session state via a helper rather than going through the login flow in every test.

```ruby
# test/test_helper.rb
def sign_in(user)
  post session_path, params: { email: user.email, password: "password" }
end
```

### Running tests

```bash
bin/rails test                                      # all tests
bin/rails test test/models/                         # one directory
bin/rails test test/models/order_test.rb            # one file
bin/rails test test/models/order_test.rb:15         # one test by line
bin/rails test:system                               # system tests separately
```

## Checklist Per Cycle

```
[ ] Test describes behavior, not implementation
[ ] Test uses public interface only
[ ] Test would survive internal refactor
[ ] Fixture used (not inline object construction where a fixture exists)
[ ] Code is minimal for this test
[ ] No speculative features added
```
