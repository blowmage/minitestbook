Chapter 3 - Minitest::Spec DSL
==============================

In chapter 1 we looked at the Minitest API and using `Minitest::Test` to create *test classes* and *test methods* using *assertion methods* and *support methods*. This style of testing may seem primitive to some who prefer using a high level spec DSL instead of an API. Minitest also provides a spec DSL, but to understand the spec DSL you must first understand the Minitest API. So I encourage you to read chapter 1 if you skipped it in your excitement.

Using a spec DSL is also very popular with practitioners of BDD, a variation of TDD. In BDD the spec DSL is used to create tests that can read as a software specification. Minitest's spec DSL is built on top of Minitest's API and provides this abstraction with minimal DIFFERENCE(?).

To demonstrate the spec DSL, let's start with the `test_hello_world.rb` file created from chapters 1 and 2. We will convert this file from using the Minitest API to the spec DSL. Here are the contents of the file we are starting with:

```ruby
gem "minitest"
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def setup
    @minibot = Minibot.new
  end

  def test_add_rule
    @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"

    assert_equal 1, @minibot.triggers.size
    assert_includes @minibot.triggers, "zomg"
  end

  def test_add_rules
    @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    @minibot.add "wtf", "http://i.imgur.com/ULQl7.gif"

    assert_equal 2, @minibot.triggers.size
    assert_includes @minibot.triggers, "zomg"
    assert_includes @minibot.triggers, "wtf"
  end

  def test_add_multiple_rules_on_same_trigger
    @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    @minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
    @minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"

    assert_equal 1, @minibot.triggers.size
    assert_includes @minibot.triggers, "zomg"
  end

  def test_triggers_is_empty
    assert_equal 0, @minibot.triggers.size
  end
end
``` `test/test_minibot.rb`

The `describe` Block
--------------------

The spec DSL is comprised of a series of nested blocks. Instead of creating a *test class* explicitly, we can use a **`describe` block** to do this for us. We can replace the line `class TestHelloWorld < Minitest::Test` with a describe:

### Before

```ruby
class TestMinibot < Minitest::Test
```

### After

```ruby
describe Minibot do
```

The `describe` block can be passed the constant of the class you are testing, or a string, or any object that can be converted to a string, like a symbol. But for the most part a class constant it preferred when possible.

We can use nested `describe` blocks to narrow the focus of our tests. This focus can be about a specific method under test, or it can be a scenario such as collaborator being valid or invalid, or permissions being present or not present. In our case, most of our *test methods* are specific to the `add` method, so let's create a nested `describe` block that is focused on that method:

```ruby
gem "minitest"
require "minitest/autorun"

describe Minibot do
  def setup
    @minibot = Minibot.new
  end

  describe :add do
    def test_add_rule
      @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"

      assert_equal 1, @minibot.triggers.size
      assert_includes @minibot.triggers, "zomg"
    end

    def test_add_rules
      @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
      @minibot.add "wtf", "http://i.imgur.com/ULQl7.gif"

      assert_equal 2, @minibot.triggers.size
      assert_includes @minibot.triggers, "zomg"
      assert_includes @minibot.triggers, "wtf"
    end

    def test_add_multiple_rules_on_same_trigger
      @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
      @minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
      @minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"

      assert_equal 1, @minibot.triggers.size
      assert_includes @minibot.triggers, "zomg"
    end
  end

  def test_triggers_is_empty
    assert_equal 0, @minibot.triggers.size
  end
end
``` `test/test_minibot.rb`

Under the covers Minitest is creating a *test class* with the contents of each `describe` block. The *test class* inherits from the `describe` block it is nested under, or the default `Minitest::Spec` class if it is not nested. So the *test class* created by the `describe :add` block inherits from the *test class* created by the `describe Minibot` block, which inherits from `Minitest::Spec` (which inherits from `Minitest::Test`).

The `describe` block is just another way to create a *test class* in your tests. The output of the spec DSL are objects using the Minitest API.

The `it` Block
--------------

You can create *test methods* using the spec DSL, just as you can create *test classes*. *Test methods* are created using the **`it` block**, which must be nested inside of a `describe` block, the same way a *test method* must be located inside a *test class*. The `it` block takes a string that can provide more descriptive description (?!?) than the name of the *test method*:

### Before

```ruby
def test_add_rule
```

```ruby
def test_add_rules
```

```ruby
def test_add_multiple_rules_on_same_trigger
```

### After

```ruby
it "adds a single rule" do
```

```ruby
it "adds multiple rules with different triggers" do
```

```ruby
it "adds several rules all on the same trigger" do
```

Our revised test is now:

```ruby
gem "minitest"
require "minitest/autorun"

describe Minibot do
  def setup
    @minibot = Minibot.new
  end

  describe :add do
    it "adds a single rule" do
      @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"

      assert_equal 1, @minibot.triggers.size
      assert_includes @minibot.triggers, "zomg"
    end

    it "adds multiple rules with different triggers" do
      @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
      @minibot.add "wtf", "http://i.imgur.com/ULQl7.gif"

      assert_equal 2, @minibot.triggers.size
      assert_includes @minibot.triggers, "zomg"
      assert_includes @minibot.triggers, "wtf"
    end

    it "adds several rules all on the same trigger" do
      @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
      @minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
      @minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"

      assert_equal 1, @minibot.triggers.size
      assert_includes @minibot.triggers, "zomg"
    end
  end

  it "has an empty trigger list by default" do
    assert_equal 0, @minibot.triggers.size
  end
end
``` `test/test_minibot.rb`

Under the covers Minitest is creating a *test method* that uses the string provided to the `it` block. So the contents of the `it "adds a single rule"` block is used to create a method named "`test_0001_adds a single rule`" and contents of the `it "adds multiple rules with different triggers"` block is used to create a method named "`test_0002_adds multiple rules with different triggers`".

The `it` block is just another way to create a *test methods* in your tests. The output of the spec DSL are methods using the Minitest API.

Expectations
------------

Instead of passing calculated values to the main `assert` method, or using special case assertions such as `assert_equal` or `assert_match`, we can use call the assertions using an alternate API(?) called **expectations**.

The common order of arguments for assertions is expected, then actual. Consider the argument names for the `assert_equal` method as defined in Minitest:

    def assert_equal exp, act, msg = nil
      msg = message(msg, "") { diff exp, act }
      assert exp == act, msg
    end

In the assertion method the position of the arguments is important, but expectations are called on the actual value eliminating the need to remember the positioning. In our tests, the `assert_equal` assertion:

```ruby
assert_equal 1, @minibot.triggers.size
```

Can be written with the `must_equal` expectation:

```ruby
@minibot.triggers.size.must_equal 1
```

Our revised test is now:

```ruby
gem "minitest"
require "minitest/autorun"

describe Minibot do
  def setup
    @minibot = Minibot.new
  end

  describe :add do
    it "adds a single rule" do
      @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"

      @minibot.triggers.size.must_equal 1
      @minibot.triggers.must_include "zomg"
    end

    it "adds multiple rules with different triggers" do
      @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
      @minibot.add "wtf", "http://i.imgur.com/ULQl7.gif"

      @minibot.triggers.size.must_equal 2
      @minibot.triggers.must_include "zomg"
      @minibot.triggers.must_include "wtf"
    end

    it "adds several rules all on the same trigger" do
      @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
      @minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
      @minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"

      @minibot.triggers.size.must_equal 1
      @minibot.triggers.must_include "zomg"
    end
  end

  it "has an empty trigger list by default" do
    @minibot.triggers.size.must_equal 0
  end
end
``` `test/test_minibot.rb`

Under the covers Minitest is calling *assertion methods* from the *expectations*. *Expectations* are just another way to call *assertion methods* in your tests. The output of the spec DSL are methods using the Minitest API.

The `before` and `after` Blocks
---------------------------

The spec DSL has analogs for the `setup` and `teardown` support methods. They are expressed as `before` and `after` blocks. The contents of the `setup` and `teardown` support methods will transfer directly to the `before` and `after` blocks.

### Before

```ruby
def setup
  @minibot = Minibot.new
end
```
### After

```ruby
before do
  @minibot = Minibot.new
end
```

The `let` Block
---------------

Since the test was only using the `setup` and `teardown` methods to initialize an instance variable in our *test class*, the spec DSL provides a way to declare this outside of the the `before` and `after` blocks. This is the `let` block.

The `let` block will create a support method that returns the result of the given block. And it will execute the block only once per test. So while it can be called several times in a test the block will only be executed the first time. And unlike the `setup` method or the `before` block it is lazy evaluated so if it is never called the block will never be invoked.

Under the covers the `let` block Minitest is creates a memoized accessor method that clears the value between tests. So the following test-style code:

```ruby
def setup
  @foo = nil
end

def foo
  @foo ||= Foo.new
end

def teardown
  @foo = nil
end
```

Can be replaced with the spec-style code:

```ruby
let(:foo) { Foo.new }
```

We can replace our use of the `before` block with a `let` block and then update all instances of the instance variable with calls to the accessor method within the test methods.

### Before

```ruby
before do
  @minibot = Minibot.new
end
```
### After

```ruby
let(:minibot) { Minibot.new }
```

This is one of the advantages (???) `let` blocks can call other let block, because they just 

The `before` and `after` and `let` blocks are more ways to create a *support methods* in your tests. The output of the spec DSL are methods using the Minitest API.

Conclusion
----------

Our revised test is now:

```ruby
gem "minitest"
require "minitest/autorun"

describe Minibot do
  let(:minibot) { Minibot.new }

  describe :add do
    it "adds a single rule" do
      minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"

      minibot.triggers.size.must_equal 1
      minibot.triggers.must_include "zomg"
    end

    it "adds multiple rules with different triggers" do
      minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
      minibot.add "wtf", "http://i.imgur.com/ULQl7.gif"

      minibot.triggers.size.must_equal 2
      minibot.triggers.must_include "zomg"
      minibot.triggers.must_include "wtf"
    end

    it "adds several rules all on the same trigger" do
      minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
      minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
      minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"

      minibot.triggers.size.must_equal 1
      minibot.triggers.must_include "zomg"
    end
  end

  it "has an empty trigger list by default" do
    minibot.triggers.size.must_equal 0
  end
end
``` `test/test_minibot.rb`

You've now converted your test from the Minitest API to the spec DSL. 

You've created *test classes* using **`describe` blocks**, created *test methods* using **`it` blocks**, called *assertion methods* using **expectations**, and created *support methods* using **`before`** and **`after`** and **`let` blocks**.

The spec DSL allows for an english-like/high-level abstraction for creating objects and methods using the Minitest API. Minitest tests can use a combination of the spec DSL and the Minitest API. You can use `it` blocks with *test classes*, and *assertions* with `let` blocks. The spec DSL generates objects and methods according to the Minitest API that Minitest uses.
