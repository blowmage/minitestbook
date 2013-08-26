TDD Cycle: Make it Red
======================

**NOTE**: We want to go from red to green as quickly as possible. We should make changes so small that we can be red for only a moment, and quickly turn it around and be back to green. Green is our safe zone. We want to stay there.

**NOTE**: How do we approach the difference between note writing any code until you have a failing test vs. being free to make any refactorings as long as the tests stay green? How do we know when we can refactor vs. adding new tests?


I've often been asked, "How is writing code with tests any different than just writing code?". This is one of my favorite questions to answer, and by the end of this chapter you will be able to answer it too.

Now that we are familiar with the Minitest API, let's take a look at how we can use Minitest to help us design our code. We will be using a practice called Test Driven Development (TDD) and the TDD Cycle. TDD is not intended to be used for quality assurance of our software; TDD is intended to be used to help us drive the design of our code. This means we want to write our tests before we write our code, and use the tests to inform what code we write.

If this doesn't make sense to you, don't worry, we will step through the TDD Cycle several times before we're done.

Step 1: Make it Red
-------------------

Let's start by adding a new *test method* for the first TODO: "Add a new matching rule". When writing this new test method we have a decision to make: What do we want the Minibot API to be for adding a new rule? What information do we nee dto provide to add the rule? How do we know that a rule was added? We need to put ourselves in the mindset of those that consume our library: how would we want to use this code? This process is key: by consuming the code first we get to change our mindset and consider what the most appropriate API should be. We are not conscerned with implementation details, we are performing a *design* excercise.

We know decide that for Minibot to register a new rule the client needs to provide a trigger for the rule (the text "zomg"), and the message to be returned (the text "http://i.imgur.com/49ORL0o.gif") when the rule is triggered. Let's pass both of those values to an `add` method.

```ruby
minibot = Minibot.new
minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
```

But how can we tell the rule was added? We aren't making any assertions, which means if this functionality fails we have no way to tell. Let's add a check that the `minibot` object knows about the trigger by asserting that the trigger size is 1.


```ruby
minibot = Minibot.new
minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
assert_equal 1, minibot.triggers.size
```

Here is what our file looks like now.

```ruby
require "helper"

class TestMinibot < Minitest::Test
  def test_add_rule
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    assert_equal 1, minibot.triggers.size
  end
end
```

`test/test_minibot.rb`

So far we have only changed our *test class*, so we expect our tests to fail:

```shell
$ ruby minibot.rb
Run options: --seed 43987

E

Finished in 0.000808s, 1237.6238 runs/s, 0.0000 assertions/s.

1) Error:
TestMinibot#test_add_rule:
NameError: uninitialized constant TestMinibot::Minibot
minibot.rb:6:in `test_add_rule'

1 runs, 0 assertions, 0 failures, 1 errors, 0 skips
```

`terminal`

Interestingly, instead of generating a failure it generated an error. The error occurred because our new test method is trying to create an new object from a class that does not exist. This is okay, Minitest expects errors to happen and will display the appropriate information.

Handling errors is part of our design process. We want to wrote tests to use our code before we write our code. And we want to make this test pass by writing as little code as possible. Let's start by fixing the 'uninitialized constant TestMinibot::Minibot' error by adding a class definition for Minibot:

```ruby
class Minibot

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

`test/test_minibot.rb`

Okay, let's rerun the tests and see if that helps:

```shell
1) Error:
TestMinibot#test_add_rule:
NoMethodError: undefined method `add' for #<Minibot:0x007fc224eb2100>
minibot.rb:11:in `test_add_rule'

1 runs, 0 assertions, 0 failures, 1 errors, 0 skips
```

Now we have a different error, this time because the test is calling a method that does not exist. The Minibot constant exists, but the `add` method does not. Let's add the method, but not the implementation, and rerun the test:

```ruby
class Minibot
  def add(trigger, response)

  end
end
```

```shell
1) Error:
TestMinibot#test_add_rule:
NoMethodError: undefined method `triggers' for #<Minibot:0x007fb494eb1ea0>
minibot.rb:14:in `test_add_rule'

1 runs, 0 assertions, 0 failures, 1 errors, 0 skips
```

Another undefined method. This time for `triggers` that we are going to use to verify that the rule was added successfully. Our test is going to ask the object returned from `triggers` for its size, but let's not implement that now since that isn't the error that Minitest is telling us about. Let's add an empty method to clear this error:

```ruby
class Minibot
  def add(trigger, response)

  end

  def triggers

  end
end
```

```shell
1) Error:
TestMinibot#test_add_rule:
NoMethodError: undefined method `size' for nil:NilClass
minibot.rb:18:in `test_add_rule'

1 runs, 0 assertions, 0 failures, 1 errors, 0 skips
```

As we expected, another error. It may not seem like such a small test method could generate this many errors, but it can and should. If we had started implementation without tests who knows what we would be implementing now. There is safety in letting the tests drive our implementation because we are only implementing what we know needs to be added.

Let's add an object that responds to `size` from the `Minibot#triggers` method to clear this error. We should expect that `triggers` is going to return a list of triggers, so let's return an empty array and see if our output changes.

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

`test/test_minibot.rb`

```shell
$ ruby minibot.rb
Run options: --seed 55576

F

Finished in 0.026938s, 37.1223 runs/s, 37.1223 assertions/s.

1) Failure:
TestMinibot#test_add_rule [minibot.rb:18]:
Expected: 1
Actual: 0

1 runs, 1 assertions, 1 failures, 0 errors, 0 skips
```

`terminal`

Ahh, now we have our first failure. This is considered *Red*. Before we had *errors*; classes or methods that were called by did not exist. Now we have a *failure*, the code's behavior does not meet the test's expectations. In this case, the test expected a value of 1, but got 0 instead. We want our tests to drive our code from error to failure to pass. Let's fix this failure and get our test passing.
