Chapter 7: Minitest Spec DSL
============================

In chapter 1 we looked at the Minitest API and using `Minitest::Test` to create *test classes* and *test methods* using *assertion methods* and *support methods*. This style of testing may seem primitive to some who prefer using a high level spec DSL instead of an API. Minitest also provides a spec DSL, but to understand the spec DSL you must first understand the Minitest API. So I encourage you to read chapter 1 if you skipped it in your excitement.

Using a spec DSL is also very popular with practitioners of BDD, a variation of TDD. In BDD the spec DSL is used to create tests that can read as a software specification. Minitest's spec DSL is built on top of Minitest's API and provides this abstraction with minimal DIFFERENCE(?).

To demonstrate the spec DSL, let's start with the recently refactored `test/test_minibot.rb` file. We will convert this file from using the Minitest API to the spec DSL. Here are the contents of the file we are starting with:

```ruby
require "minitest/autorun"
require "minibot"

class TestMinibot < Minitest::Test
  def setup
    @minibot        = Minibot.new
    @zomg_responses = [ "http://i.imgur.com/49ORL0o.gif",
                        "http://i.imgur.com/KqCfQPR.gif",
                        "http://i.imgur.com/fc5LmfO.gif" ]
    @wtf_response   =   "http://i.imgur.com/ULQl7.gif"

    @minibot.add "zomg", @zomg_responses[0]
    @minibot.add "zomg", @zomg_responses[1]
    @minibot.add "zomg", @zomg_responses[2]
    @minibot.add "wtf",  @wtf_response
  end

  def test_match_zomg
    response = @minibot.match "ZOMG! This is cool!"
    assert_includes @zomg_responses, response
  end

  def test_match_wtf
    response = @minibot.match "Wait, this works? WTF?"
    assert_equal @wtf_response, response
  end

  def test_match_returns_random_response
    sample = 300.times.map { @minibot.match "ZOMG! This is cool!" }
    responses = Hash[sample.group_by{|i| i }.map{|k,v| [k,v.size]}]

    assert_in_delta 100, responses[@zomg_responses[0]], 33
    assert_in_delta 100, responses[@zomg_responses[1]], 33
    assert_in_delta 100, responses[@zomg_responses[2]], 33
  end

  def test_no_match
    response = @minibot.match "ZOINKS! This doesn't match!"
    assert_nil response
  end

  def test_adding_rules
    assert_equal 2, @minibot.triggers.size
    assert_includes @minibot.triggers, "zomg"
    assert_includes @minibot.triggers, "wtf"
  end

  def test_adding_no_rules
    @minibot = Minibot.new
    assert_equal 0, @minibot.triggers.size
    refute_includes @minibot.triggers, "zomg"
    refute_includes @minibot.triggers, "wtf"
  end

  def test_zomg_responses
    responses = @minibot.responses "zomg"
    assert_equal 3, responses.size
    assert_includes responses, @zomg_responses[0]
    assert_includes responses, @zomg_responses[1]
    assert_includes responses, @zomg_responses[2]
  end

  def test_wtf_responses
    responses = @minibot.responses "wtf"
    assert_equal 1, responses.size
    assert_includes responses, @wtf_response
  end

  def test_no_responses
    responses = @minibot.responses "zoinks"
    assert_equal 0, responses.size
  end
end
```

`test/test_minibot.rb`

The `describe` Block
--------------------

The spec DSL is comprised of a series of nested blocks. Instead of creating a *test class* explicitly, we can use a **`describe` block** to do this for us. We can replace the class definition with a describe:

### Before

```ruby
class TestMinibot < Minitest::Test
```

### After

```ruby
describe Minibot do
```

The `describe` block can be passed the constant of the class you are testing, or a string, or any object that can be converted to a string, like a symbol. But for the most part a class constant it preferred when possible.

We can use nested `describe` blocks to narrow the focus of our tests. This focus can be about a specific method under test, or it can be a scenario such as collaborator being valid or invalid, or permissions being present or not present. For instance, there are several *test methods* specific to the `match`, `add`, and `responses` methods. Therefire we can create nested `describe` blocks that contain those tests:

```ruby
require "minitest/autorun"
require "minibot"

describe Minibot do
  def setup
    @minibot        = Minibot.new
    @zomg_responses = [ "http://i.imgur.com/49ORL0o.gif",
                        "http://i.imgur.com/KqCfQPR.gif",
                        "http://i.imgur.com/fc5LmfO.gif" ]
    @wtf_response   =   "http://i.imgur.com/ULQl7.gif"

    @minibot.add "zomg", @zomg_responses[0]
    @minibot.add "zomg", @zomg_responses[1]
    @minibot.add "zomg", @zomg_responses[2]
    @minibot.add "wtf",  @wtf_response
  end

  describe :match do
    def test_match_zomg
      response = @minibot.match "ZOMG! This is cool!"
      assert_includes @zomg_responses, response
    end

    def test_match_wtf
      response = @minibot.match "Wait, this works? WTF?"
      assert_equal @wtf_response, response
    end

    def test_match_returns_random_response
      sample = 300.times.map { @minibot.match "ZOMG! This is cool!" }
      responses = Hash[sample.group_by{|i| i }.map{|k,v| [k,v.size]}]

      assert_in_delta 100, responses[@zomg_responses[0]], 33
      assert_in_delta 100, responses[@zomg_responses[1]], 33
      assert_in_delta 100, responses[@zomg_responses[2]], 33
    end

    def test_no_match
      response = @minibot.match "ZOINKS! This doesn't match!"
      assert_nil response
    end
  end

  describe :add do
    def test_adding_rules
      assert_equal 2, @minibot.triggers.size
      assert_includes @minibot.triggers, "zomg"
      assert_includes @minibot.triggers, "wtf"
    end

    def test_adding_no_rules
      @minibot = Minibot.new
      assert_equal 0, @minibot.triggers.size
      refute_includes @minibot.triggers, "zomg"
      refute_includes @minibot.triggers, "wtf"
    end
  end

  describe :responses do
    def test_zomg_responses
      responses = @minibot.responses "zomg"
      assert_equal 3, responses.size
      assert_includes responses, @zomg_responses[0]
      assert_includes responses, @zomg_responses[1]
      assert_includes responses, @zomg_responses[2]
    end

    def test_wtf_responses
      responses = @minibot.responses "wtf"
      assert_equal 1, responses.size
      assert_includes responses, @wtf_response
    end

    def test_no_responses
      responses = @minibot.responses "zoinks"
      assert_equal 0, responses.size
    end
  end
end
```

`test/test_minibot.rb`

Under the covers Minitest is creating a *test class* with the contents of each `describe` block. The *test class* inherits from the `describe` block it is nested under, or the default `Minitest::Spec` class if it is not nested. So the *test class* created by the `describe :add` block inherits from the *test class* created by the `describe Minibot` block, which inherits from `Minitest::Spec` (which inherits from `Minitest::Test`).

The `describe` block is just another way to create a *test class* in your tests. The output of the spec DSL are objects using the Minitest API.

The `it` Block
--------------

You can create *test methods* using the spec DSL, just as you can create *test classes*. *Test methods* are created using the **`it` block**, which must be nested inside of a `describe` block, the same way a *test method* must be located inside a *test class*. The `it` block takes a string that can provide more expressive description than the name of the *test method*. In our test:

```ruby
def test_match_zomg
```

Can be written with an `it` block:

```ruby
it "matches the trigger 'zomg'" do
```

### Before

```ruby
describe :match do
  def test_match_zomg
    response = @minibot.match "ZOMG! This is cool!"
    assert_includes @zomg_responses, response
  end

  def test_match_wtf
    response = @minibot.match "Wait, this works? WTF?"
    assert_equal @wtf_response, response
  end

  def test_match_returns_random_response
    sample = 300.times.map { @minibot.match "ZOMG! This is cool!" }
    responses = Hash[sample.group_by{|i| i }.map{|k,v| [k,v.size]}]

    assert_in_delta 100, responses[@zomg_responses[0]], 33
    assert_in_delta 100, responses[@zomg_responses[1]], 33
    assert_in_delta 100, responses[@zomg_responses[2]], 33
  end

  def test_no_match
    response = @minibot.match "ZOINKS! This doesn't match!"
    assert_nil response
  end
end
```

### After

```ruby
describe :match do
  it "matches the trigger 'zomg'" do
    response = @minibot.match "ZOMG! This is cool!"
    assert_includes @zomg_responses, response
  end

  it "matches the trigger 'wtf'" do
    response = @minibot.match "Wait, this works? WTF?"
    assert_equal @wtf_response, response
  end

  it "returns a random response" do
    sample = 300.times.map { @minibot.match "ZOMG! This is cool!" }
    responses = Hash[sample.group_by{|i| i }.map{|k,v| [k,v.size]}]

    assert_in_delta 100, responses[@zomg_responses[0]], 33
    assert_in_delta 100, responses[@zomg_responses[1]], 33
    assert_in_delta 100, responses[@zomg_responses[2]], 33
  end

  it "doesn't match an unknown trigger" do
    response = @minibot.match "ZOINKS! This doesn't match!"
    assert_nil response
  end
end
```

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

### Before

```ruby
describe :match do
  it "matches the trigger 'zomg'" do
    response = @minibot.match "ZOMG! This is cool!"
    assert_includes @zomg_responses, response
  end

  it "matches the trigger 'wtf'" do
    response = @minibot.match "Wait, this works? WTF?"
    assert_equal @wtf_response, response
  end

  it "returns a random response" do
    sample = 300.times.map { @minibot.match "ZOMG! This is cool!" }
    responses = Hash[sample.group_by{|i| i }.map{|k,v| [k,v.size]}]

    assert_in_delta 100, responses[@zomg_responses[0]], 33
    assert_in_delta 100, responses[@zomg_responses[1]], 33
    assert_in_delta 100, responses[@zomg_responses[2]], 33
  end

  it "doesn't match an unknown trigger" do
    response = @minibot.match "ZOINKS! This doesn't match!"
    assert_nil response
  end
end
```

### After

```ruby
describe :match do
  it "matches the trigger 'zomg'" do
    response = @minibot.match "ZOMG! This is cool!"
    @zomg_responses.must_include response
  end

  it "matches the trigger 'wtf'" do
    response = @minibot.match "Wait, this works? WTF?"
    response.must_equal @wtf_response
  end

  it "returns a random response" do
    sample = 300.times.map { @minibot.match "ZOMG! This is cool!" }
    responses = Hash[sample.group_by{|i| i }.map{|k,v| [k,v.size]}]

    responses[@zomg_responses[0]].must_be_close_to 100, 33
    responses[@zomg_responses[1]].must_be_close_to 100, 33
    responses[@zomg_responses[2]].must_be_close_to 100, 33
  end

  it "doesn't match an unknown trigger" do
    response = @minibot.match "ZOINKS! This doesn't match!"
    response.must_be_nil
  end
end
```

`test/test_minibot.rb`

Under the covers Minitest is calling *assertion methods* from the *expectations*. *Expectations* are just another way to call *assertion methods* in your tests. The output of the spec DSL are methods using the Minitest API.

The `before` and `after` Blocks
---------------------------

The spec DSL has analogs for the `setup` and `teardown` support methods. They are expressed as `before` and `after` blocks. The contents of the `setup` and `teardown` support methods will transfer directly to the `before` and `after` blocks.

### Before

```ruby
def setup
  @minibot        = Minibot.new
  @zomg_responses = [ "http://i.imgur.com/49ORL0o.gif",
                      "http://i.imgur.com/KqCfQPR.gif",
                      "http://i.imgur.com/fc5LmfO.gif" ]
  @wtf_response   =   "http://i.imgur.com/ULQl7.gif"

  @minibot.add "zomg", @zomg_responses[0]
  @minibot.add "zomg", @zomg_responses[1]
  @minibot.add "zomg", @zomg_responses[2]
  @minibot.add "wtf",  @wtf_response
end
```
### After

```ruby
before do
  @minibot        = Minibot.new
  @zomg_responses = [ "http://i.imgur.com/49ORL0o.gif",
                      "http://i.imgur.com/KqCfQPR.gif",
                      "http://i.imgur.com/fc5LmfO.gif" ]
  @wtf_response   =   "http://i.imgur.com/ULQl7.gif"

  @minibot.add "zomg", @zomg_responses[0]
  @minibot.add "zomg", @zomg_responses[1]
  @minibot.add "zomg", @zomg_responses[2]
  @minibot.add "wtf",  @wtf_response
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
  @minibot        = Minibot.new
  @zomg_responses = [ "http://i.imgur.com/49ORL0o.gif",
                      "http://i.imgur.com/KqCfQPR.gif",
                      "http://i.imgur.com/fc5LmfO.gif" ]
  @wtf_response   =   "http://i.imgur.com/ULQl7.gif"

  @minibot.add "zomg", @zomg_responses[0]
  @minibot.add "zomg", @zomg_responses[1]
  @minibot.add "zomg", @zomg_responses[2]
  @minibot.add "wtf",  @wtf_response
end
```
### After

```ruby
let(:minibot)        { Minibot.new }
let(:zomg_responses) { [ "http://i.imgur.com/49ORL0o.gif",
                         "http://i.imgur.com/KqCfQPR.gif",
                         "http://i.imgur.com/fc5LmfO.gif" ] }
let(:wtf_response)   { "http://i.imgur.com/ULQl7.gif" }

before do
  minibot.add "zomg", zomg_responses[0]
  minibot.add "zomg", zomg_responses[1]
  minibot.add "zomg", zomg_responses[2]
  minibot.add "wtf",  wtf_response
end
```

One of the advantages of the `let` blocks is that they can call other let blocks. They can be used within the `before` and `after` blocks. And they are lazy evaluated, so if you have an object that is expensive to initialize but rarely used, you don't pay the performance penalty on the tests that don't call it.

The `before` and `after` and `let` blocks are more ways to create a *support methods* in your tests. The output of the spec DSL are methods using the Minitest API.

Final Result
------------

Our revised test is now:

```ruby
require "minitest/autorun"
require "minibot"

describe Minibot do
  let(:minibot)        { Minibot.new }
  let(:zomg_responses) { [ "http://i.imgur.com/49ORL0o.gif",
                           "http://i.imgur.com/KqCfQPR.gif",
                           "http://i.imgur.com/fc5LmfO.gif" ] }
  let(:wtf_response)   { "http://i.imgur.com/ULQl7.gif" }

  before do
    minibot.add "zomg", zomg_responses[0]
    minibot.add "zomg", zomg_responses[1]
    minibot.add "zomg", zomg_responses[2]
    minibot.add "wtf",  wtf_response
  end

  describe :match do
    it "matches the trigger 'zomg'" do
      response = minibot.match "ZOMG! This is cool!"
      zomg_responses.must_include response
    end

    it "matches the trigger 'wtf'" do
      response = minibot.match "Wait, this works? WTF?"
      response.must_equal wtf_response
    end

    it "returns a random response" do
      sample = 300.times.map { minibot.match "ZOMG! This is cool!" }
      responses = Hash[sample.group_by{|i| i }.map{|k,v| [k,v.size]}]

      responses[zomg_responses[0]].must_be_close_to 100, 33
      responses[zomg_responses[1]].must_be_close_to 100, 33
      responses[zomg_responses[2]].must_be_close_to 100, 33
    end

    it "doesn't match an unknown trigger" do
      response = minibot.match "ZOINKS! This doesn't match!"
      response.must_be_nil
    end
  end

  describe :add do
    it "adds triggers for added rules" do
      minibot.triggers.size.must_equal 2
      minibot.triggers.must_include "zomg"
      minibot.triggers.must_include "wtf"
    end

    it "defaults to empty triggers" do
      minibot = Minibot.new
      minibot.triggers.size.must_equal 0
      minibot.triggers.wont_include "zomg"
      minibot.triggers.wont_include "wtf"
    end
  end

  describe :responses do
    it "knows responses for 'zomg'" do
      responses = minibot.responses "zomg"
      responses.size.must_equal 3
      responses.must_include zomg_responses[0]
      responses.must_include zomg_responses[1]
      responses.must_include zomg_responses[2]
    end

    it "knows responses for 'wtf'" do
      responses = minibot.responses "wtf"
      responses.size.must_equal 1
      responses.must_include wtf_response
    end

    it "returns empty responses for unknown trigger" do
      responses = minibot.responses "zoinks"
      responses.size.must_equal 0
    end
  end
end
```

You've now converted your test from the Minitest API to the spec DSL. 

Recap
-----

You've created *test classes* using **`describe` blocks**, created *test methods* using **`it` blocks**, called *assertion methods* using **expectations**, and created *support methods* using **`before`** and **`after`** and **`let` blocks**.

The spec DSL allows for an english-like/high-level abstraction for creating objects and methods using the Minitest API. Minitest tests can use a combination of the spec DSL and the Minitest API. You can use `it` blocks with *test classes*, and *assertions* with `let` blocks. The spec DSL generates objects and methods according to the Minitest API that Minitest uses.
