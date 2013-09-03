TDD Cycle: Refactor Redux
=========================

We have iterated on our tests and driven the design of our code. We have refactored our code while keeping the tests passing, and without altering the tests. Our code is cleaner and better for this, but what about our tests?

Far too often the criticisms I hear about tests are that the tests themselves are too difficult or time-consuming to maintain. Your tests are part of your application's codebase. They have a cost the same as the rest of your code. Bad tests can cost more than the benefit they provide.

I think I know why this happens. It is very easy to think of your tests as write-only code. Code that is not revisited after it is first created. But nothing can be further from the truth. Your tests need as much care and thought put into them as the rest of your code. This is why I like to take a moment after refactoring my code using passing tests to refactor my tests using well designed code.

Step 3: Make it Right Again
---------------------------

Now that we are happy with the design and implementation of Minibot, we can take a look at the tests. Here is the state of our tests up to this point:

```ruby
require "minitest/autorun"
require "minibot"

class TestMinibot < Minitest::Test
  def test_match
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    response = minibot.match "ZOMG! This is cool!"
    assert_equal "http://i.imgur.com/49ORL0o.gif", response
  end

  def test_match_returns_random_response
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
    minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"

    sample = 300.times.map { minibot.match "ZOMG! This is cool!" }
    responses = Hash[sample.group_by{|i| i }.map{|k,v| [k,v.size]}]

    assert_in_delta 100, responses["http://i.imgur.com/49ORL0o.gif"], 33
    assert_in_delta 100, responses["http://i.imgur.com/KqCfQPR.gif"], 33
    assert_in_delta 100, responses["http://i.imgur.com/fc5LmfO.gif"], 33
  end

  def test_no_match
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    response = minibot.match "ZOINKS! This doesn't match!"
    assert_nil response
  end

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

  def test_responses
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    minibot.add "zomg", "http://i.imgur.com/KqCfQPR.gif"
    minibot.add "zomg", "http://i.imgur.com/fc5LmfO.gif"
    minibot.add "wtf",  "http://i.imgur.com/ULQl7.gif"

    responses = minibot.responses "zomg"
    assert_equal 3, responses.size
    assert_includes responses, "http://i.imgur.com/49ORL0o.gif"
    assert_includes responses, "http://i.imgur.com/KqCfQPR.gif"
    assert_includes responses, "http://i.imgur.com/fc5LmfO.gif"

    responses = minibot.responses "wtf"
    assert_equal 1, responses.size
    assert_includes responses, "http://i.imgur.com/ULQl7.gif"
  end
end
```

`test/test_minibot.rb`

There is a good amount of duplication in these tests. Enough that I want to spend a little time revising the design of the tests so that the next person that works in this code has an easier time understanding the expectations. Especially if the next person is a future me. If you to the TDD Cycle right, you will be revisiting and updating your tests *more* than you change your code.

### Implement `setup` method

The majority of the duplication I see has to do with test setup. We are adding the same rules in nearly every test. I wonder if we could find a common set of rules for all our tests and set it in the `setup` method. How about creating a Minibot instance with the 3 "zomg" rules and the one "wtf" rule?

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

### Improved `match` tests

Now let's turn our attention to the `Minibot#match` tests. We have a match, a no match, and a random response test. We can change the match test to look for any response in the `@zomg_responses` array, and add another test to match the "wtf" rule. Here is what those tests can look like using the shared setup.

```ruby
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

  assert_in_delta 100, responses[@zomg_responses[0]], 25
  assert_in_delta 100, responses[@zomg_responses[1]], 25
  assert_in_delta 100, responses[@zomg_responses[2]], 25
end

def test_no_match
  response = @minibot.match "ZOINKS! This doesn't match!"
  assert_nil response
end
```

Much better! The tests are clearer and to the point. Most don't have *Arrange* steps, and provide the *Act* and *Assert* steps only. While there is some data duplication in the tests, notably the message string sent to `Minibot#match`, I'm happy with the behavior they describe. And best of all the tests still pass cleanly:

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 47256

# Running:

........

Finished in 0.001475s, 5423.7288 runs/s, 18983.0508 assertions/s.

8 runs, 28 assertions, 0 failures, 0 errors, 0 skips
```

### Improved `add` tests

With the shared Minibot instance, the `add` tests can be consolidated. We can also add some additional coverage, asserting that the triggers are empty when no rules have been added.

```ruby
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
```

### Improved `responses` tests

The `responses` tests can be simplified as well. We can simply assert that the data entered in `setup` is the same data that is available from the Minibot object. We can add additional coverage like we did for `add`, asserting that the responses for an unadded trigger returns an empty list (and doesn't raise an exception).

```ruby
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
```

Again, we should only revisit the test design after all code under test is properly refactored while passing all the tests cleanly.

Recap
-----

The *Refactor* step provides an opportunity to revisit the design of all our code, both the application code and the tests we use to drive it's design. Don't be afraid of making changes to your tests. Take care of your tests and they will take care of you.

