Chapter 2 - TDD Cycle
=====================

**NOTE**: We want to go from red to green as quickly as possible. We should make changes so small that we can be red for only a moment, and quickly turn it around and be back to green. Green is our safe zone. We want to stay there.

**NOTE**: How do we approach the difference between note writing any code until you have a failing test vs. being free to make any refactorings as long as the tests stay green? How do we know when we can refactor vs. adding new tests?


I've often been asked, "How is writing code with tests any different than just writing code?". This is one of my favorite questions to answer, and by the end of this chapter you will be able to answer it too.

Now that we are familiar with the Minitest API, let's take a look at how we can use Minitest to help us design our code. We will be using a practice called Test Driven Development (TDD) and the TDD Cycle. TDD is not intended to be used for quality assurance of our software; TDD is intended to be used to help us drive the design of our code. This means we want to write our tests before we write our code, and use the tests to inform what code we write.

If this doesn't make sense to you, don't worry, we will step through the TDD Cycle several times before we're done.

Simple Requirements
-------------------

Why did i choose "minibot" in the previous chapter? Because I love bots, all kids of bots. IRC bots. Campfire bots. Twitter bots. They make smile (for the most part). I want to build a bot that is well designed, and I'm going to use Minitest to help me get there.

Since I don't want to get lots in the weeds of all the things that a bot can do, let's start simple. Here is a TODO list of the things I want this bot to do. We will work off this list and add to it as we progress.

```plain
Minibot
=======

A simple bot for storing and matching memes.

- [ ] Add a new matching rule
- [ ] List all matching rules
- [ ] Remove a matching rule
- [ ] Match an given string to an existing rule
``` `todo.md`

New Feature
-----------

Let's add one of the features from our TODO. I want to register a rule that a message can match. So any messages that include the word "zomg" in them would get an appropriate animated gif response. If you sent the message "zomg this is amazing!", the bot would respond "http://i.imgur.com/49ORL0o.gif".

We are going to write a test *before* we write the code. This is known as test-first, as opposed to test-after. TDD is a test-first approach, while QA is typically a test-after approach. For the remainder of the book we will be writing tests before writing code.

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
gem "minitest"
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def test_add_rule
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    assert_equal 1, minibot.triggers.size
  end
end
``` `minibot.rb`

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
``` `terminal`

Interestingly, instead of generating a failure it generated an error. The error occurred because our new test method is trying to create an new object from a class that does not exist. This is okay, Minitest expects errors to happen and will display the appropriate information.

Handling errors is part of our design process. We want to wrote tests to use our code before we write our code. And we want to make this test pass by writing as little code as possible. Let's start by fixing the 'uninitialized constant TestMinibot::Minibot' error by adding a class definition for Minibot:

```ruby
class Minibot

end

gem "minitest"
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def test_add_rule
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    assert_equal 1, minibot.triggers.size
  end
end
``` `minibot.rb`

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

gem "minitest"
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def test_add_rule
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    assert_equal 1, minibot.triggers.size
  end
end
``` `minibot.rb`

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
``` `terminal`

Ahh, now we have our first failure. This is considered *Red*. Before we had *errors*; classes or methods that were called by did not exist. Now we have a *failure*, the code's behavior does not meet the test's expectations. In this case, the test expected a value of 1, but got 0 instead. We want our tests to drive our code from error to failure to pass. Let's fix this failure and get our test passing.

Step 2: Make it Green
---------------------

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

gem "minitest"
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def test_add_rule
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    assert_equal 1, minibot.triggers.size
  end
end
``` `minibot.rb`

What an awful solution! Surely Minitest wouldn't allow something as bad as that to pass, right?

```shell
$ ruby minibot.rb
Run options: --seed 32365

.

Finished in 0.000784s, 1275.5102 runs/s, 1275.5102 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
``` `terminal`

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

class Minibot
  def add(trigger, response)
    @triggers ||= []
    @triggers << trigger
  end

  def triggers
    @triggers
  end
end

  1) Failure:
TestMinibot#test_add_multiple_rules_on_same_trigger [minibot.rb:29]:
Expected: 1
  Actual: 3

2 runs, 3 assertions, 1 failures, 0 errors, 0 skips

This makes the `test_add_rules` test pass, but now the `test_add_multiple_rules_on_same_trigger` test fails. We are still *Red*. We are failing because now we have duplicate triggers in the list. The quickest way to get the test passing and us back into *green* is to return a list of unique triggers.

def triggers
  @triggers.uniq
end

With that change we can run our tests again and see that they are all passing.

$ ruby minibot.rb
Run options: --seed 8669

# Running:

..

Finished in 0.001366s, 1464.1288 runs/s, 2196.1933 assertions/s.

2 runs, 3 assertions, 0 failures, 0 errors, 0 skips


We let the tests drive our code implementation and we are *Green* again. We are starting to describe a functional and usable API for Minibot, but I have low confidence in this code. If a client calls `triggers` before `add` then the `@triggers` list will be nil, causing an error. So let's add a failing test for that.

def test_triggers_is_empty
  minibot = Minibot.new
  assert_equal 0, minibot.triggers.size
end

  1) Error:
TestMinibot#test_triggers_is_empty:
NoMethodError: undefined method `uniq' for nil:NilClass
    minibot.rb:8:in `triggers'
    minibot.rb:34:in `test_triggers_is_empty'

3 runs, 3 assertions, 0 failures, 1 errors, 0 skips

Again, our goal is to be *Green* as quickly as possible. Part of me wants to write some code to initailize `@triggers` in the initializer, but My Worse Programmer mindset needs to win here. The fastest way to make this test pass is to add some duplicate code to the `triggers` method and instantiate the list there. The speed of writing one line of bad code wins out over writing three lines of better code.

def triggers
  @triggers ||= []
  @triggers.uniq
end


$ ruby minibot.rb
Run options: --seed 11535

# Running:

...

Finished in 0.001904s, 1575.6303 runs/s, 2100.8403 assertions/s.

3 runs, 4 assertions, 0 failures, 0 errors, 0 skips


How do you feel about the code you've written? If you are anything like me then you probably don't feel very good about it. You've possibly been shouting down the voice inside of your head complaining about duplicate code and design patterns. The truth is, you should feel like this. The Worst Programmer mindset takes all the shortcuts I've spend my career trying to avoid. This style of programming makes me feels like I'm trolling myself. (Or trolling my pair programmer, which is *so very fun*.)

The Customer mindset wants to ensure that the API works in the "Make it Red" step, but the Worst Programmer wants to ensure that we spend as little time being *Red* as possible, taking any and every shortcut to get back to *Green*. The result is a very rapid feedback cycle on the design if the API that you are providing at the cost of your code quality.









Awful. Does it pass?

    $ ruby -r minitest/autorun minibot.rb
    Run options: --seed 47356

    # Running:

    .....

    Finished in 0.001368s, 3654.9708 runs/s, 3654.9708 assertions/s.

    5 runs, 5 assertions, 0 failures, 0 errors, 0 skips
{terminal}

It should feel like you are trolling yourself. The Consumer mindset excersizes your creativity by finding the edges of what is acceptable behavior for your API, while the Worst Programmer mindset excercises your cretical thinking by takeing every short cut possible to keep the code passing the tests. The result is a suite of tests that describe the API. The focus is all on API design and test coverage. The tests enforce the design the API. Now we are ready to move on to the final phase of the TDD Cycle.

Step 3: Make it Right
---------------------

Now that the Customer and Worst Programmer mindsets have gone rounds we have a suite of tests that demonstrate the API and show how it is intended to work. Now we can focus on changing the code so that it is correct. This step is also known as the *Refactoring* step, because we want to be able to make changes to how our code is implemented without changing the API we have defined. Let's switch to the Engineer mindset and start thinking about what improvements we should make to the code. Luckilly, we have some good test coverage so that we will know if we change the expected behavior the next time we run the tests.

Right now HelloWorld#hello has five conditional branches. My Engineer mindset doesn't like that. How can we change the implementation without changing the API? We could use a different default value and use string interpolation.

```ruby
class Minibot
  def initialize
    @rules = Hash.new { |hash, key| hash[key] = [] }
  end

  def add(trigger, response)
    @rules[trigger] << response
  end

  def triggers
    @rules.keys
  end
end
``` `minibot.rb`

Much better! Does it continue to pass?

```shell
$ ruby minibot.rb
Run options: --seed 64926

# Running:

...

Finished in 0.001264s, 2373.4177 runs/s, 3164.5570 assertions/s.

3 runs, 4 assertions, 0 failures, 0 errors, 0 skips
``` `terminal`

Yep, it passes and all is right with the world. If I had ideas of design patterns to use I would implement them now. But the key in this phase is to only change the code, not the tests. If you change the tests in this phase you are redesigning and not refactoring.

*NOTE*: Tests need to be written and rewritten. Far too often I see tests that were written once and never revisited again. If you to the TDD Cycle right, you will be revisiting and updateing your tests *more* than you change your code.

I said that the Refactoring step was to make changes to the code, and that's true. But once we have all the code right its a good idea to revisit the design of the tests. We have a fair bit of duplication in our tests, like creating a Minibot object for each test method. We can extract that out to a `setup` method.

```ruby
 class TestMinibot < Minitest::Test
  def setup
    @minibot = Minibot.new
  end

  def test_add_rules
    @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    assert_equal 1, @minibot.triggers.size
    @minibot.add "wtf", "http://i.imgur.com/ULQl7.gif"
    assert_equal 2, @minibot.triggers.size
  end

  def test_add_multiple_rules_on_same_trigger
    @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    @minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
    @minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"
    assert_equal 1, @minibot.triggers.size
  end

  def test_triggers_is_empty
    assert_equal 0, @minibot.triggers.size
  end
end
```

The other thing that stands out to me is that we only checking the size of the triggers, and not the contents. We can probably add some more test coverage around the intended behavior of the triggers. I think it would be sufficent to state that the triggers list includes the trigger that was added.


```ruby
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
```

Much better! While there is some data duplication across some of the tests, I'm happy with the behavior they describe. And best of all the tests still pass cleanly:

```shell
$ ruby minibot.rb
Run options: --seed 42383

# Running:

....

Finished in 0.000963s, 4153.6864 runs/s, 12461.0592 assertions/s.

4 runs, 12 assertions, 0 failures, 0 errors, 0 skips
``` `terminal`

Again, we only let our Engineering mindset consider the test design after all code under test is properly refactored, while all the tests continue to pass cleanly. And by delaying our Engineering mindset until the *Refactoring* step we have given our Consumer mindset the ability to design the optimum API.


Conclusion
----------

You've now written tests to go from **Error** to **Failure** to **Passing** through TDD's **Red**, **Green, **Refactor** steps. The dirty little secret is that you will to *Red* *Green* *Red* *Green* *Red* *Green* and then *Refactor*. In this process the tests provide sufficent coverage for the API you design for your code's consumers. You then use the feedback your tests provide to make changes. The end result is you will have the confidence to make improvements to your software because you enjoy the assurance that you aren't breaking behavior.

The TDD Cycle also promotes looking at your code from difference mindsets. You look at your code from multiple perspectives, from the Consumer to the Engineer, from the Designer to the Maintainer, allowing you to entertain both your creativity and your focus/completeness/something (!!). The more you practice TDD, the better at these mindsets you get, and the better code you will write.
