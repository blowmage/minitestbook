TDD Cycle: Make it Green
========================

???

Worst Programmer Mindset
------------------------

When we were writing our tests we put ourselves in the mindset of the consumers of our code. We thought through what API we wanted to use, not just what was easy to expose. Now that we are changing our code I want to change mindset once again. I want to put on the personality of the Worst Programmer in the World. I want to take any and all shortcuts needed to get the test to pass. I want to intentionally write bad or buggy code.

How would the Worst Programmer in the World make this test pass? Probably hard code the implementation so it matches the expectations of the test. So let's do that by making the `triggers` method return an object that has a size of 1. How about an array with a single nil value?

```ruby
class Minibot
  def add(trigger, response)

  end

  def triggers
    []
  end
end

require "helper"

class TestMinibot < Minitest::Test
  def test_add_rule
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    assert_equal 1, minibot.triggers.size
  end
end
```

`minibot.rb`

What an awful solution! Surely Minitest wouldn't allow something as bad as that to pass, right?

```shell
$ ruby minibot.rb
Run options: --seed 32365

.

Finished in 0.000784s, 1275.5102 runs/s, 1275.5102 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

`terminal`

It passes! Our terrible code passes! Passing is considered *Green*. But the code feels dirty, doesn't it? The `add` method doesn't do anything and the `triggers` method is hardcoded to the test expectation. Surely this isn't how we want our code to be.

Time for another mindset change. We want to return to our Consumer Mindset and write more tests that flesh out all more of the expected behavior. Can we add multiple rules? Can we add multiple rules on the same trigger? Let's find out by writing some more tests. First, let's add to our existing test methods and add a second rule and a second trigger size assertion.

```ruby
def test_add_rules
  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
  assert_equal 1, minibot.triggers.size
  minibot.add "wtf", "http://i.imgur.com/ULQl7.gif"
  assert_equal 2, minibot.triggers.size
end
```

Let's also add a second test method that adds multiple rules on the same trigger, and makes sure the fact that all these rules share the same trigger is reflected in the trigger size.

```ruby
def test_add_multiple_rules_on_same_trigger
  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
  minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
  minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"
  assert_equal 1, minibot.triggers.size
end
```

Running the tests now puts us back into *Red*.

```shell
  1) Failure:
TestMinibot#test_add_rules [minibot.rb:20]:
Expected: 2
  Actual: 1

2 runs, 3 assertions, 1 failures, 0 errors, 0 skips
```

The `test_add_multiple_rules_on_same_trigger` test passes, but `test_add_rules` doesn't. We want to get back to *Green* quckly. How can we do it? The way the tests are written we'll have to actually implement some logic here. We could save the trigger sent to `add` in a list and return that from the `triggers` method. Let's try that to get this test passing.

```ruby
class Minibot
  def add(trigger, response)
    @triggers ||= []
    @triggers << trigger
  end

  def triggers
    @triggers
  end
end
```

```shell
  1) Failure:
TestMinibot#test_add_multiple_rules_on_same_trigger [minibot.rb:29]:
Expected: 1
  Actual: 3

2 runs, 3 assertions, 1 failures, 0 errors, 0 skips
```

This makes the `test_add_rules` test pass, but now the `test_add_multiple_rules_on_same_trigger` test fails. We are still *Red*. We are failing because now we have duplicate triggers in the list. The quickest way to get the test passing and us back into *green* is to return a list of unique triggers.

```ruby
def triggers
  @triggers.uniq
end
```

With that change we can run our tests again and see that they are all passing.

```shell
$ ruby minibot.rb
Run options: --seed 8669

# Running:

..

Finished in 0.001366s, 1464.1288 runs/s, 2196.1933 assertions/s.

2 runs, 3 assertions, 0 failures, 0 errors, 0 skips
```

We let the tests drive our code implementation and we are *Green* again. We are starting to describe a functional and usable API for Minibot, but I have low confidence in this code. If a client calls `triggers` before `add` then the `@triggers` list will be nil, causing an error. So let's add a failing test for that.

```ruby
def test_triggers_is_empty
  minibot = Minibot.new
  assert_equal 0, minibot.triggers.size
end
```

```shell
  1) Error:
TestMinibot#test_triggers_is_empty:
NoMethodError: undefined method `uniq' for nil:NilClass
    minibot.rb:8:in `triggers'
    minibot.rb:34:in `test_triggers_is_empty'

3 runs, 3 assertions, 0 failures, 1 errors, 0 skips
```

Again, our goal is to be *Green* as quickly as possible. Part of me wants to write some code to initailize `@triggers` in the initializer, but My Worse Programmer mindset needs to win here. The fastest way to make this test pass is to add some duplicate code to the `triggers` method and instantiate the list there. The speed of writing one line of bad code wins out over writing three lines of better code.

```ruby
def triggers
  @triggers ||= []
  @triggers.uniq
end
```

```shell
$ ruby minibot.rb
Run options: --seed 11535

# Running:

...

Finished in 0.001904s, 1575.6303 runs/s, 2100.8403 assertions/s.

3 runs, 4 assertions, 0 failures, 0 errors, 0 skips
```

How do you feel about the code you've written? If you are anything like me then you probably don't feel very good about it. You've possibly been shouting down the voice inside of your head complaining about duplicate code and design patterns. The truth is, you should feel like this. The Worst Programmer mindset takes all the shortcuts I've spend my career trying to avoid. This style of programming makes me feels like I'm trolling myself. (Or trolling my pair programmer, which is *so very fun*.)

The Customer mindset wants to ensure that the API works in the "Make it Red" step, but the Worst Programmer wants to ensure that we spend as little time being *Red* as possible, taking any and every shortcut to get back to *Green*. The result is a very rapid feedback cycle on the design if the API that you are providing at the cost of your code quality.

It should feel like you are trolling yourself. The Consumer mindset excersizes your creativity by finding the edges of what is acceptable behavior for your API, while the Worst Programmer mindset excercises your cretical thinking by takeing every short cut possible to keep the code passing the tests. The result is a suite of tests that describe the API. The focus is all on API design and test coverage. The tests enforce the design the API. Now we are ready to move on to the final phase of the TDD Cycle.
