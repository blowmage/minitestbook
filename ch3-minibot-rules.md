Minibot: Adding Rules
=====================

Our goal in TDD is to move from *Red* to *Green* as quickly as possible. To move from the *Consumer* to *worst Programmer* minsets as quickly as possible. The failing test gives us guidance and direction on what to implement, and the passing test allows us to explore new areas of behavior for our code.

To illustrate this let's revisit our TODO. We can check off the initial task, but in doing so we found it neccessary to have a way to add a rule. Let's add a new task for adding a rule, and one for deleting a rule. We don't know if we will ever need to delete a rule, but we might so its a good idea to commit that to the list.

```plain
Minibot
=======

A simple bot for storing and matching memes.

- [x] Match a message string to a rule
- [ ] Add a rule
- [ ] Remove a rule
```

`todo.md`

Add a rule
----------

How do we test that a rule has been added? We know a rule is a "trigger", that when matched will return a "response". So how do we verify that a rule was added? We probably need some way of asking Minibot what triggers have been added. How does this work? What if multiple rules are added for the same trigger? What if different rules are added for different triggers?

Let's write a few tests that cover those possibilities. Minibot should return a list of the triggers added to it. Multiple rules can be added to the same trigger. And Minibot can have more than one rule added to it. Here is how these tests can be implemented:

```ruby
def test_adding_one_rule
  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"

  assert_equal 1, minibot.triggers.size
  assert_includes minibot.triggers, "zomg"
end

def test_adding_multiple_rules
  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
  minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
  minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"

  assert_equal 1, minibot.triggers.size
  assert_includes minibot.triggers, "zomg"
end

def test_adding_different_rules
  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
  minibot.add "wtf",  "http://i.imgur.com/ULQl7.gif"

  assert_equal 2, minibot.triggers.size
  assert_includes minibot.triggers, "zomg"
  assert_includes minibot.triggers, "wtf"
end
```

Unfortunately, all three tests fail when run.

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 41171

.EE.E

Finished in 0.000937s, 5336.1793 runs/s, 2134.4717 assertions/s.

1) Error:
TestMinibot#test_adding_multiple_rules:
NoMethodError: undefined method `triggers' for #<Minibot:0x007f8e4bed2368>
test/test_minibot.rb:33:in `test_adding_multiple_rules'

2) Error:
TestMinibot#test_adding_one_rule:
NoMethodError: undefined method `triggers' for #<Minibot:0x007f8e4bed12d8>
test/test_minibot.rb:23:in `test_adding_one_rule'

3) Error:
TestMinibot#test_adding_different_rules:
NoMethodError: undefined method `triggers' for #<Minibot:0x007f8e4bed12d8>
test/test_minibot.rb:42:in `test_adding_different_rules'

5 runs, 2 assertions, 0 failures, 3 errors, 0 skips
```

Our goal is to be *Green* as quickly as possible, but we can only focus on one test at a time. To help us focus on ust one test, let's skip the latter test by calling `skip` within them:

```ruby
def test_adding_multiple_rules
  skip "Skipping adding multiple for now"

  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
  minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
  minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"

  assert_equal 1, minibot.triggers.size
  assert_includes minibot.triggers, "zomg"
end

def test_adding_different_rules
  skip "Skipping adding different for now"

  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
  minibot.add "wtf",  "http://i.imgur.com/ULQl7.gif"

  assert_equal 2, minibot.triggers.size
  assert_includes minibot.triggers, "zomg"
  assert_includes minibot.triggers, "wtf"
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 14649

ESS..

Finished in 0.000963s, 5192.1080 runs/s, 2076.8432 assertions/s.

1) Error:
TestMinibot#test_adding_one_rule:
NoMethodError: undefined method `triggers' for #<Minibot:0x007f9643ed31a0>
test/test_minibot.rb:23:in `test_adding_one_rule'

5 runs, 2 assertions, 0 failures, 1 errors, 2 skips
```

Now we can focus on getting `test_adding_one_rule` passing. This test is failing because it is making a call to `Minibot#triggers`, which doesn't exist yet. Let's add this method and rerun the tests:

```ruby
class Minibot
  def add trigger, response
    # TODO: Save this data
  end

  def match message
    # TODO: Refactor. Response is coupled to test.
    "http://i.imgur.com/49ORL0o.gif" if message == "ZOMG! This is cool!"
  end

  def triggers
    # TODO: Return all the added triggers
  end
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 8097

# Running:

ESS..

Finished in 0.000993s, 5035.2467 runs/s, 2014.0987 assertions/s.

1) Error:
TestMinibot#test_adding_one_rule:
NoMethodError: undefined method `size' for nil:NilClass
test/test_minibot.rb:23:in `test_adding_one_rule'

5 runs, 2 assertions, 0 failures, 1 errors, 2 skips
```

As we may have expected, another error. It may feel uncomfortable to generate this many errors, but this is an important part of TDD. We want feedback so we know what to do next. Feedback on what our code is doing gives us confidence. If we had started implementation without tests who knows what we would be implementing now. There is safety in letting the tests drive our implementation because we are only implementing what we know needs to be added.

Let's add an object that responds to `size` from the `Minibot#triggers` method to clear this error. We should expect that `triggers` is going to return a list of triggers, so let's return an empty array and see if our test output changes.

```ruby
def triggers
  # TODO: Return all the added triggers
  []
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 25885

# Running:

S..FS

Finished in 0.027610s, 181.0938 runs/s, 108.6563 assertions/s.

1) Failure:
TestMinibot#test_adding_one_rule [test/test_minibot.rb:23]:
Expected: 1
Actual: 0

5 runs, 3 assertions, 1 failures, 0 errors, 2 skips
```

We have gone from *Error* to *Failure*. We know our Minibot implementation now implements the structure of the API our tests are describing, now we need to implement the behavior as well. Our tests expect the trigger "zomg" to be returned, so let's return it. Again, the Worst Programmer mindset says the fastest way to get this test to pass is to hardcode the implementation.

```ruby
def triggers
  # TODO: Return all the added triggers
  ["zomg"]
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 51218

# Running:

.S..S

Finished in 0.001060s, 4716.9811 runs/s, 4716.9811 assertions/s.

5 runs, 5 assertions, 0 failures, 0 errors, 2 skips
```

Success! We are back to *Green*! Let's remove the `skip` call from the `test_adding_multiple_rules` test and see if it also passes:

```ruby
def test_adding_multiple_rules
  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
  minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
  minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"

  assert_equal 1, minibot.triggers.size
  assert_includes minibot.triggers, "zomg"
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 2163

# Running:

S....

Finished in 0.000962s, 5197.5052 runs/s, 8316.0083 assertions/s.

5 runs, 8 assertions, 0 failures, 0 errors, 1 skips
```

It passes and our Worst Programmer mindset is happy because there are no changes needed. Let;s remove the last `skip` from the `test_adding_different_rules` test. Looking at the implementation I expect this test to fail.

```ruby
def test_adding_different_rules
  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
  minibot.add "wtf",  "http://i.imgur.com/ULQl7.gif"

  assert_equal 2, minibot.triggers.size
  assert_includes minibot.triggers, "zomg"
  assert_includes minibot.triggers, "wtf"
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 49489

F....

Finished in 0.020623s, 242.4478 runs/s, 436.4060 assertions/s.

1) Failure:
TestMinibot#test_adding_different_rules [test/test_minibot.rb:42]:
Expected: 2
Actual: 1

5 runs, 9 assertions, 1 failures, 0 errors, 0 skips
```

The goal is to quickly get back to Green. But here is where I struggle. I want to take a few minites and think through the possibilities for how to implement this. My Engineering mindset starts to take over. But you need to push that away. Now is not the time for thinking about implementation. We are focusing on the Consumer API and using the rapid Red/Green feedback to help us design that API. There is no time for contemplating implementation now. So with a renewed focus let's figure out how to get the `test_adding_different_rules` test to pass as quickly as possible.

We know we want to return only the triggers sent to Minibot. So let's store the triggers in an `@triggers` instance variable and return them in the `Minibot#triggers` method:


```ruby
class Minibot
  def add trigger, response
    @triggers ||= []
    @triggers << trigger
  end

  def match message
    # TODO: Refactor. Response is coupled to test.
    "http://i.imgur.com/49ORL0o.gif" if message == "ZOMG! This is cool!"
  end

  def triggers
    @triggers
  end
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 38859

# Running:

.F...

Finished in 0.019181s, 260.6746 runs/s, 573.4842 assertions/s.

1) Failure:
TestMinibot#test_adding_multiple_rules [test/test_minibot.rb:33]:
Expected: 1
Actual: 3

5 runs, 11 assertions, 1 failures, 0 errors, 0 skips
```

This change gets the `test_adding_different_rules` test passing, but now the `test_adding_multiple_rules` test is failing. It is expecting one trigger for all three rules added. Again, how can we get this test passing with the least amount of code? We could return only unique triggers.

```ruby
def triggers
  @triggers.uniq
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 4569

.....

Finished in 0.000948s, 5274.2616 runs/s, 13713.0802 assertions/s.

5 runs, 13 assertions, 0 failures, 0 errors, 0 skips
```

With these tests we have specified the expected behavior for adding rules to Minibot. And we have discovered more behavior around exposing a list of triggers that have been added to Minibot.

Recap
-----

How do you feel about the code you've written? If you are anything like me then you probably don't feel very good about it. You've possibly been shouting down the voice inside of your head complaining about duplicate code and design patterns. The truth is, you should feel like this. The Worst Programmer mindset takes all the shortcuts I've spend my career trying to avoid. This style of programming makes me feels like I'm trolling myself. (Or trolling my pair when pair programming, which is *so very fun*.)

We have a few more features to design before we are done with Minibot's design.
