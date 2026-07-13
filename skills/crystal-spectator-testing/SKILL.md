---
name: crystal-spectator-testing
description: >
  Create and update Crystal language tests using the Spectator testing framework.
  Use whenever a user wants to write new tests, update existing specs, or migrate
  Crystal specs from the built-in Spec to Spectator in their Crystal shard.
  Spectator is a more capable testing framework with RSpec-like syntax, full
  mocking/doubles support, and rich matchers. Use this skill anytime Crystal
  testing code is involved, especially when mocking, stubbing, or writing
  comprehensive specs.
---

# Spectator Testing for Crystal

Spectator is a BDD testing framework for Crystal, modeled after RSpec. It is fully compatible with Crystal's built-in Spec and adds significantly more capability, especially in the area of mocking and doubles.

## Setting Up Spectator

Add Spectator as a development dependency to your shard's `shard.yml`:

```yaml
development_dependencies:
  spectator:
    gitlab: arctic-fox/spectator
```

Then run `shards` to install it, and create or update `spec/spec_helper.cr`:

```crystal
require "../src/*"
require "spectator"
# Optional: require "spectator/should" # if you want the should syntax
```

If your code previously used `require "spec"`, remove it — loading both frameworks together causes conflicts.

## Spec Structure

All specs must be wrapped in a top-level `Spectator.describe` block. Nested `describe` and `context` blocks use the unprefixed forms.

```crystal
require "./spec_helper"

Spectator.describe String do
  context "when empty" do
    describe "#empty?" do
      it "returns true" do
        expect("".empty?).to be_true
      end
    end
  end

  context "when not empty" do
    describe "#empty?" do
      it "returns false" do
        expect("foo".empty?).to be_false
      end
    end
  end
end
```

### Context Types

- **`describe`** — Use when describing a class, method, or return value.
- **`context`** — Use when explaining a situation or scenario.
- **`it` / `specify`** — An individual test. The description is optional but recommended.
- **`pending` / `skip` / `xit`** — Skipped tests (stub for incomplete features).

### Concise Test Patterns

**`provided`** — Condenses repetitive tests with different inputs into a single block per input. Each block is a separate example:

```crystal
subject(user) { User.new(age) }

provided age: 10 do
  expect(user.can_drive?).to be_false
  expect(user.can_vote?).to be_false
end

provided age: 18 do
  expect(user.can_drive?).to be_true
  expect(user.can_vote?).to be_true
end
```

**`sample`** — Runs all nested tests repeatedly with different values:

```crystal
sample [1, 2, 3, 4, 5] do |int|
  it "accepts the value" do
    expect(int).to be_a(Int32)
  end
end

# Or with a named block argument omitted:
sample [-10..10], count: 5 do
  it "accepts a value" do
    expect(value).to be_between(-10, 10)
  end
end
```

**`random_sample`** — Same as `sample` but randomly selects values (count is required).

## Test Values: `let`, `subject`, and `is_expected`

### `let` — Lazy, Cached Definitions

`let` defines a named value that is lazy-evaluated (first access triggers evaluation) and cached within the same test. Values are NOT shared across tests.

```crystal
let(user) { User.new("Bob") }
let(post) { Post.new(title: "Hello") }

it "has the name" do
  expect(user.name).to eq("Bob")
end
```

Nested contexts can override outer `let` definitions:

```crystal
Spectator.describe User do
  let(age) { 25 }

  context "when minor" do
    let(age) { 16 }

    it { expect(user.age).to eq(16) }
  end
end
```

### `let!` — Forced Evaluation

`let!` evaluates the value *before* each test (instead of lazily). Useful when a test depends on side effects like database inserts.

```crystal
let!(user) { User.create(name: "Bob") }

it "saves to database" do
  expect(User.find_by(name: "Bob")).to_not be_nil
end
```

### `subject` — Defining the Thing Under Test

`subject` defines the object being tested. It is lazy, cached, and accessible across nested contexts.

```crystal
Spectator.describe String do
  subject { "foobar" }

  describe "#upcase" do
    it "returns uppercase" do
      expect(subject.upcase).to eq("FOOBAR")
    end
  end
end
```

**Named subjects** allow multiple subjects in the same context (only the last one is accessible as `subject`):

```crystal
subject(array1) { [1, 2, 3] }
subject(array2) { [4, 5, 6] }

it "are different" do
  expect(array1).to_not eq(array2)
end
```

### `is_expected` — Shorthand for `expect(subject)`

When `subject` is defined, use `is_expected` to shorten expectations:

```crystal
subject { "foobar" }
is_expected.to start_with("foo")
is_expected.to_not end_with("bar")
```

### `described_class` — Implicit Reference to the Top-Level Type

When `Spectator.describe` wraps a type, `described_class` refers to it, and an implicit `subject { described_class.new }` is created.

```crystal
Spectator.describe User do
  describe "#initialize" do
    it "creates a guest by default" do
      expect(subject.guest?).to be_true
    end
  end

  describe "#initialize with args" do
    it "sets the name" do
      user = subject(name: "Bob")
      expect(user.name).to eq("Bob")
    end
  end
end
```

### `super` — Modifying a Nested Subject

Use `super` to derive from an outer subject:

```crystal
Spectator.describe String do
  subject { "foo" }

  describe "#upcase" do
    subject { super.upcase }

    it { is_expected.to eq("FOO") }
  end
end
```

## Helper Methods (Quick Reference)

Helper methods defined within a context are available to that context and all nested ones. They can also access `let` values:

```crystal
Spectator.describe String do
  let(length) { 10 }

  def random_string
    chars = ('a'..'z').to_a
    String.build(length) { |b| length.times { b << chars.sample } }
  end

  describe "#size" do
    subject { random_string.size }
    it { is_expected.to eq(10) }
  end
end
```

Helper methods can also be defined as modules and included:

```crystal
module StringHelpers
  def random_string
    chars = ('a'..'z').to_a
    String.build(10) { |b| 10.times { b << chars.sample } }
  end
end

Spectator.describe String do
  include StringHelpers
  # ...
end
```
## Expectations & Matchers

An expectation is `expect(actual).to(MATCHER)` or `expect(actual).to_not(MATCHER)` (also `not_to`). For a block that may raise, use `expect { block }.to(MATCHER)`.

### Equality Matchers

| Matcher | Description | Example |
|---------|-------------|---------|
| `eq` | `==` comparison | `expect(foo).to eq(bar)` |
| `ne` | `!=` comparison | `expect(foo).to ne(bar)` |
| `be` | Same reference (`.same?`) for class instances; value comparison for structs | `expect(arr).to be(same_arr)` |
| `match` | `===` / semantic equality | `expect(tuple).to match({Int32, String})` |

### Comparison Matchers

| Matcher | Description | Example |
|---------|-------------|---------|
| `be_lt` / `be <` | Less than | `expect(5).to be_lt(10)` |
| `be_le` / `be <=` | Less than or equal | `expect(5).to be_le(5)` |
| `be_gt` / `be >` | Greater than | `expect(10).to be_gt(5)` |
| `be_ge` / `be >=` | Greater than or equal | `expect(5).to be_ge(5)` |
| `be_within(delta).of(center)` | Within tolerance (uses `includes?`) | `expect(pi).to be_within(0.01).of(3.14)` |
| `be_close(expected, delta)` | Same as `be_within` but with Crystal's default Spec syntax | `expect(pi).to be_close(3.14, 0.01)` |
| `be_between(min, max)` | Within a range (inclusive by default; add `.inclusive` or `.exclusive`) | `expect(5).to be_between(1, 10).exclusive` |

### Type Matchers

| Matcher | Description | Example |
|---------|-------------|---------|
| `be_a(T)` / `be_an(T)` | Checks `is_a?(T)` | `expect("foo").to be_a(String)` |
| `respond_to(method)` | Checks method existence | `expect(42).to respond_to(:even?)` |

### String Matchers

| Matcher | Description | Example |
|---------|-------------|---------|
| `contain(str, ...)` / `have(str, ...)` | Checks for substring(s) existence | `expect("foobar").to contain("foo", "bar")` |
| `contain_elements([strs])` / `have_elements([strs])` | Same as `contain` but takes an array | `expect("foobar").to contain_elements(["foo", "bar"])` |
| `match(regex)` | Regex match (`=~`) | `expect("foo").to match(/foo\|bar/)` |
| `start_with(str/char/regex)` | Checks prefix | `expect("foo").to start_with("f")` |
| `end_with(str/char/regex)` | Checks suffix | `expect("foo").to end_with("o")` |
| `be_empty` | Checks if string is empty | `expect("").to be_empty` |

### Collection Matchers

| Matcher | Description | Example |
|---------|-------------|---------|
| `contain(items...)` | At least those items exist (uses `includes?`) | `expect([1,2,3]).to contain(1, 3)` |
| `contain_elements([items])` | Array form of `contain` | |
| `have(criteria...)` | Items match via `===` (types, regex, values) | `expect([1,2,3]).to have(1, String)` |
| `have_elements([criteria])` | Array form of `have` | |
| `contain_exactly(items...).in_order` | Exact same items in order | `expect([3,1,2]).to contain_exactly(1,2,3).in_any_order` |
| `match_array(array)` | Same as `contain_exactly` but takes an array | `expect([3,1,2]).to match_array([1,2,3]).in_any_order` |
| `start_with(criteria)` | First element matches via `===` | `expect([1,2,3]).to start_with(1)` |
| `end_with(criteria)` | Last element matches via `===` | `expect([1,2,3]).to end_with(3)` |
| `be_empty` | Empty collection | `expect([] of Int32).to be_empty` |
| `have_key(key)` / `has_key(key)` | Hash/NamedTuple has a key | `expect({a: 1}).to have_key(:a)` |
| `have_value(val)` / `has_value(val)` | Hash/NamedTuple has a value | `expect({a: 1}).to have_value(1)` |
| `all(matcher)` | Every element satisfies the matcher | `expect([1,2,3]).to all(be_a(Int32))` |

### Truthy Matchers

| Matcher | Description |
|---------|-------------|
| `be` / `be_truthy` | Not `false` and not `nil` |
| `be_true` | Exactly `true` |
| `be_false` / `be_falsey` | Exactly `false` or `nil` |
| `be_nil` | Exactly `nil` |

### Error Matchers

| Matcher | Description | Example |
|---------|-------------|---------|
| `raise_error` | Block raises an error | `expect { raise "oops" }.to raise_error` |
| `raise_error(Type)` | Raises a specific error type | `expect { arr[99] }.to raise_error(IndexError)` |
| `raise_error(Type, msg)` | Raises a specific type and message pattern | `expect { arr[99] }.to raise_error(/bounds/i)` |

The Crystal default syntax `expect_raises { block }` is also available.

### Predicate Matchers

Auto-generated from any `*?` predicate method on the subject:

```crystal
expect("").to be_empty        # checks String#empty?
expect("foo").to be_ascii_only # checks String#ascii_only?
```

Prepend the predicate name (without `?`) with `be_`.

### Other Matchers

| Matcher | Description | Example |
|---------|-------------|---------|
| `have_attributes(**kwargs)` | Checks multiple attributes/methods at once | `expect(person).to have_attributes(name: "Bob", age: 30)` |
| `change { expr }` | Checks that an expression changes after executing a block | `expect { i += 1 }.to change { i }.by(1)` |

Modifiers for `change`: `.from(before)`, `.to(after)`, `.by(amount)`, `.by_at_least(n)`, `.by_at_most(n)`.

## Hooks

### `before_each` / `after_each`

Run before/after every example in the context and nested contexts. Can access `let` values.

```crystal
before_each { DB.populate_with_sample_data }
after_each { DB.empty }

it "works" do
  # DB has sample data
end
```

### `before_all` / `after_all`

Run once before/after all examples in a context (and nested). Cannot access `let` values. Use when setup/teardown is expensive.

### `around_each`

Wraps each example with a proc:

```crystal
around_each do |proc|
  DB.transaction do
    proc.call
  end
end
```

### `pre_condition` / `post_condition`

Run before/after each example like expectations, but fail the example if they don't pass. Useful for invariant checks:

```crystal
let(original) { array.dup }
pre_condition { expect(array).to_not be_nil }
post_condition { expect(array).to match_array(original) }
```

Hooks of the same kind run in definition order. With nesting, outer `before` hooks run first; outer `after` hooks run last.

## Deferred & Stub Expectations

### Deferring Expectations

Check after hooks have run:

```crystal
after_each { array << 1 }

it "pushes an element" do
  expect(array).to_eventually eq([1])
end

# Or:
expect(actual).to_never(some_matcher)
expect(actual).never_to(some_matcher)
```

### Stubs (Defining Behavior)

```crystal
dbl = double(:my_double)
allow(dbl).to receive(:answer).and_return(42)
expect(dbl.answer).to eq(42)
```

Key stub modifiers:
- **`and_return(val1, val2, ...)`** — Returns successive values until exhausted, then returns the last one.
- **`and_raise(Type, msg)`** — Raises an exception.
- **`with(arg1, arg2)`** — Constrains stub to specific arguments.
- **`receive(:method) { block }`** — Proc stub: block is evaluated to get return value.

### Spy Expectations (Verifying Interactions)

```crystal
dbl = double(:target)
subject.call_target(dbl)
expect(dbl).to have_received(:call_method).with(42).once
```

Count modifiers: `once`, `twice`, `exactly(n).times`, `at_least(n).times`, `at_most(n).times`, `at_least_once`, `at_most_once`, `at_least_twice`, `at_most_twice`.

### Expect-Receive Shortcut

```crystal
dbl = double(:target)
expect(dbl).to receive(:call).and_return(42)
dbl.call  # stubbed AND verified as called
```

## Mocks, Doubles & Null Objects

### Doubles (Duck-Typed Stand-Ins)

Define once, instantiate many times. Doubles respond to any method, but raise `UnexpectedMessage` for un-stubbed methods.

```crystal
double :user, name: "Anonymous" do
  stub def active?
    true
  end
end

it "creates a user" do
  user = double(:user)
  expect(user.name).to eq("Anonymous")
end
```

Keyword arguments define default stubs: `double(:user, name: "Bob", age: 30)`.

Block-based stubs require exact argument signatures:
```crystal
stub def process(count, limit: 10)
  count <= limit
end
```

### Anonymous Doubles

Quick one-off doubles without a prior definition:

```crystal
dbl = double(foo: 42, bar: "hello")
expect(dbl.foo).to eq(42)
```

Note: return type is the union of all provided values plus nil — don't use when Crystal requires a static return type.

### Mocks (Type-Safe Stand-Ins)

Mocks only respond to methods defined on the real type, producing compile-time errors for undefined methods. Use when you have a type restriction.

```crystal
# Given class Logger
mock Logger

it "logs a message" do
  logger = mock(Logger)
  allow(logger).to receive(:log).and_return(nil)
  logger.log("test")
end
```

For classes with required initializer arguments, override `initialize` in the mock:

```crystal
mock DB do
  def initialize(@pool = nil)  # Note: no `stub` prefix
    @pool ||= Pool.new
  end
end
```

### Null Objects

Add `.as_null_object` to any double to have undefined methods return the double itself, enabling method chaining:

```crystal
dbl = double(:user).as_null_object
expect(dbl.profile.age.country).to be_a(String)
```

### Mock Modules

Mock a module and use it with `mock(ModuleName)` or `class_mock(ModuleName)` for class methods:

```crystal
module Runnable
  def run
    "Running #{command}"
  end

  abstract def command
end

mock Runnable, command: "ls -l"
runnable = mock(Runnable)
expect(runnable.run).to eq("Running ls -l")
```

### Injecting Mocks

For concrete structs (which can't normally be mocked) or external types:

```crystal
inject_mock MyStruct, some_method: 42 do
  stub def other_method(x, y)
    x + y
  end
end

# Then just instantiate normally:
obj = MyStruct.new
```

Warning: This alters the type's behavior everywhere (not just in tests). Prefer regular mocks/doubles when possible.

## Configuration & CLI

### Runtime Configuration (in `spec_helper.cr`)

```crystal
Spectator.configure do |config|
  config.fail_fast                    # Stop on first failure
  config.fail_blank                   # Fail if no examples run
  config.dry_run                      # Show what would run
  config.randomize                    # Randomize test order
  config.seed = 12345                 # Seed for randomization
  config.profile                      # Show 10 slowest specs
  config.formatter = "json"           # Output format
  config.filter_run_including :focus  # Run only tagged :focus
  config.filter_run_excluding :slow   # Exclude :slow tagged tests
end
```

### Command-Line Options

| Flag | Description |
|------|-------------|
| `-f` / `--fail-fast` | Stop on first failure |
| `-b` / `--fail-blank` | Fail if no examples run |
| `-d` / `--dry-run` | Show what would run without executing |
| `-r` / `--rand` | Randomize test order |
| `--seed N` | Set random seed (implies `--rand`) |
| `--order ORDER` | `defined`, `rand`, or `rand:SEED` |
| `-e STRING` / `--example` | Run examples matching STRING |
| `-l LINE` / `--line` | Run test on a specific line |
| `--tag TAG` | Run/ exclude by tag; use `~` to exclude |
| `-v` / `--verbose` | Verbose document output |
| `-p` / `--profile` | Show slowest 10 specs |
| `--json` | JSON output |
| `--tap` | TAP format output |
| `--junit_output DIR` | JUnit XML output |
| `--html_output DIR` | HTML output report |
| `--no-color` | Disable colored output |

### Output Formatters

- **Default (dots):** One character per test (pass, fail, skip).
- **`--verbose` / `-v`:** Lists test names as they run.
- **`--json`:** Detailed JSON with expectation info.
- **`--tap`:** TAP (Test Anything Protocol) format.
- **`--junit_output DIR`:** JUnit XML for CI integration.
- **`--html_output DIR`:** HTML report.

## Migrating from Crystal's Built-in Spec

1. Add Spectator to `shard.yml` as shown above and run `shards`.
2. In `spec/spec_helper.cr`, replace `require "spec"` with `require "spectator"`.
3. Add `require "spectator/should"` if you need to keep the old `should` syntax.
4. Prefix every top-level `describe` block with `Spectator.`:
   ```crystal
   # Before
   describe MyClass do

   # After
   Spectator.describe MyClass do
   ```
5. Run `crystal spec` — everything should work as before.

## Best Practices

- **One spec file per type** — a spec file should typically describe one class/module.
- **Use `is_expected` when a `subject` is defined** — keeps tests concise.
- **Prefer `let` over helper methods** — lazy evaluation and caching make specs more efficient.
- **Use `before_each` over `before_all`** — each test should start with a known, isolated state.
- **Prefer `eq` over `be ==`** — `eq` captures expression labels for better failure output.
- **Use the `change` matcher for state changes** — clearer than manual before/after comparisons.
- **Mock interfaces, not implementations** — use doubles for duck-typed dependencies, mocks for type-checked ones.
- **Use `provided` to reduce repetition** — when multiple methods share the same setup inputs.

## References

For deeper dives into any topic, see the official Spectator wiki: [gitlab.com/arctic-fox/spectator/wikis/home](https://gitlab.com/arctic-fox/spectator/-/wikis/home)
