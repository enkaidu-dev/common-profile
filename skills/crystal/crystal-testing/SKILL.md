---
name: crystal-testing
description: Write test specs for Crystal programs using Crystal's built-in Spec module and DSL. Use this skill whenever you need to create unit tests, write spec files (_spec.cr), or understand how to test Crystal code with the built-in testing framework (not Spectator). This skill covers creating describe/context/it blocks, expectations, matchers, and running specs with crystal spec command.
license: MIT
---

# Crystal Built-in Testing Guide

This skill explains how to write tests for Crystal programs using the built-in `Spec` module - NOT Spectator. Make sure to use this whenever you're asked about testing Crystal code, writing unit tests, or creating `_spec.cr` files.

## Quick Start: Running Tests

Tests in Crystal are run with the `crystal spec` command. By convention, test files go in a `spec/` directory and end with `_spec.cr`.

```bash
# Run all specs
crystal spec

# Run specific spec file
crystal spec spec/my_feature_spec.cr

# Run a specific line/group
crystal spec spec/my_feature_spec.cr:42

# Filter by tags
crystal spec --tag 'fast'
```

## Anatomy of a Spec File

Every spec file needs to require the `spec` module:

```crystal
require "spec"
```

### Structure of Specs

Tests are organized using three main building blocks:

- **`describe`** - Groups related tests (typically a class or method)
- **`context`** - Sets up a specific scenario/state for the group
- **`it`** - Defines an individual test case with expected behavior

## Basic Example

Here's a complete spec file structure:

```crystal
require "spec"

# Outer describe is typically the class name
describe Array do
  # Describe instance methods with #, class methods with .
  describe "#size" do
    it "correctly reports the number of elements in the Array" do
      [1, 2, 3].size.should eq 3
    end

    it "returns zero for empty array" do
      ([] of Int32).size.should eq 0
    end
  end

  describe "#empty?" do
    context "when array has elements" do
      it "is false" do
        [1].empty?.should be_false
      end
    end

    context "when array is empty" do
      it "is true" do
        ([] of Int32).empty?.should be_true
      end
    end
  end
end
```

## Method Naming Conventions

Follow these naming patterns for clarity:

- Outer `describe`: Class name (e.g., `Array`, `MyProject::User`)
- Inner `describe` for instance methods: `"#method_name"`
- Inner `describe` for class methods: `".method_name"`
- Use `context` to describe states/scenarios

## Expectations and Matchers

### Basic Equality

```crystal
actual.should eq(expected)        # passes if actual == expected
actual.should be(expected)        # passes if actual.same?(expected) - identity check
actual.should be_a(Type)          # passes if actual.is_a?(Type)
actual.should be_nil              # passes if actual.nil?
```

### Truthiness

```crystal
actual.should be_true             # passes if actual == true
actual.should be_false            # passes if actual == false
actual.should be_truthy           # passes if actual is truthy (not nil or false)
actual.should be_falsey           # passes if actual is falsey (nil, false, or null pointer)
```

### Comparisons

```crystal
actual.should be < expected       # passes if actual < expected
actual.should be <= expected      # passes if actual <= expected
actual.should be > expected       # passes if actual > expected
actual.should be >= expected      # passes if actual >= expected
```

### Other Matchers

```crystal
actual.should contain(expected)   # passes if actual.includes?(expected)
actual.should match(pattern)      # passes if actual =~ pattern (regex/string match)
actual.should be_close(expected, delta)  # passes if (actual - expected).abs <= delta
```

## Expecting Errors

Use `expect_raises` to verify exceptions:

```crystal
# Basic exception test
expect_raises(DivisionByZeroError) do
  result = 1 / 0
end

# With message check
expect_raises(MyError, "error message") do
  # code that raises MyError with message containing "error message"
end

# With regex pattern
expect_raises(MyError, /error \w+/) do
  # code that raises MyError with message matching the pattern
end

# Capture and test the exception
ex = expect_raises(MyError) do
  # ...
end
ex.some_property.should eq expected_value
```

## Hooks for Setup and Teardown

To maintain clean and isolated tests, use hooks to set up state before tests or clean up after them.

```crystal
describe MyService do
  before do
    # Runs before each 'it' block in this describe/context
    @service = MyService.new
    @data = [1, 2, 3]
  end

  after do
    # Runs after each 'it' block
    @service = nil
    @data = []
  end

  before_all do
    # Runs once before any tests in this describe/context block
    @shared_resource = HeavyResource.new
  end

  after_all do
    # Runs once after all tests in this block have completed
    @shared_resource.close
  end

  it "performs an operation on provided data" do
    @service.process(@data).should eq [1, 2, 3]
  end
end
```

## Manual Mocking and Stubbing

Since the built-in `Spec` module does not include a powerful mocking framework like Spectator, you should use manual dependency injection or simple "Double" classes for unit testing.

### Dependency Injection
The most reliable way to test components in isolation is to inject dependencies via their constructors:

```crystal
class MyController
  def initialize(@repository)
  end

  def get_user(id)
    @repository.find(id)
  end
end

# In your spec file...
class UserRepositoryMock
  def find(id)
    User.new(id: id, name: "Mock User")
  end
end

describe MyController do
  it "returns a user from the repository" do
    mock_repo = UserRepositoryMock.new
    controller = MyController.new(mock_repo)
    
    user = controller.get_user(1)
    user.name.should eq "Mock User"
  end
end
```

### Simple Stubbing
For simpler cases, you can use a class that inherits from the real object and overrides specific methods:

```crystal
class RealService
  def fetch_data; "real data"; end
end

class StubbedService < RealService
  def fetch_data; "stubbed data"; end
end

describe MyComponent do
  it "handles stubbed data correctly" do
    service = StubbedService.new
    component = MyComponent.new(service)
    component.process.should eq "PROCESSED: stubbed data"
  end
end
```

## Focusing Tests

Mark specific tests to run only focused ones:

```crystal
it "focused test", focus: true do
  # This will ONLY run when other specs have focus: true
end
```

When any spec has `focus: true`, only those specs run. Remove focus when done.

## Tagging Specs

Add tags for filtering at runtime:

```crystal
# Single tag
it "slow integration test", tags: "integration" do
  sleep(60)
end

# Multiple tags (using Array or Set)
it "tagged test", tags: ["network", "external"] do
  # ...
end
```

Run tagged specs from command line:

```bash
crystal spec --tag 'integration'      # Run only specs with this tag
crystal spec --tag '~slow'            # Run specs NOT tagged as slow
```

## Spec Helper for Setup

Create `spec/spec_helper.cr` for common setup code that all tests use:

**spec/spec_helper.cr:**
```crystal
require "spec"
require "../src/my_project"

# Share helper methods across tests
def create_test_user(name)
  User.new(name, email: "#{name.downcase}@example.com")
end
```

**spec/user_spec.cr:**
```crystal
require "./spec_helper"

describe User do
  describe "#email" do
    it "converts name to lowercase email domain" do
      user = create_test_user("Alice")
      user.email.should eq "alice@example.com"
    end
  end
end
```

## Test Structure Template

Use this template for consistent test organization:

```
spec/
├── spec_helper.cr          # Common setup and requires
└── feature_spec.cr         # Tests for that feature/module
```

Follow this structure in each `it` block:

1. Set up the scenario (create objects, set state)
2. Execute the code being tested
3. Assert expectations using `.should` matchers
4. Clean up if needed (though specs should be isolated)

## Common Patterns and Tips

### Pending Tests

Mark tests that aren't ready yet:

```crystal
pending "feature needs implementation" do
  # Won't run, shown in reports as pending
end
```

### Context for Different States

Use `context` to test different scenarios:

```crystal
describe User do
  context "when user is active" do
    it "allows login" do
      # ...
    end
  end

  context "when user is deactivated" do
    it "prevents login" do
      # ...
    end
  end
end
```

### Multiple Expectations in One Test

You can have multiple expectations per `it` block:

```crystal
it "returns correct data structure" do
  result = my_method
  result.should be_a(ResultType)
  result.status.should eq :success
  result.data.should contain("expected")
end
```

## Running Tests by Location

Specify exactly what to run on the command line:

```bash
# All specs in spec/**/*_spec.cr (default)
crystal spec

# All specs without color output
crystal spec --no-color

# Specs matching specific directory tree
crystal spec spec/Unit/Feature/

# Specific file only
crystal spec spec/my_spec.cr

# Line number or describe/context block start
crystal spec spec/my_spec.cr:25
```

## References

For more details, see the official Crystal testing documentation at https://crystal-lang.org/reference/guides/testing.html