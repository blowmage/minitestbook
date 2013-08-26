TDD Cycle: Make it Right
========================

???

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
```

`test/test_minibot.rb`

Much better! Does it continue to pass?

```shell
$ ruby minibot.rb
Run options: --seed 64926

# Running:

...

Finished in 0.001264s, 2373.4177 runs/s, 3164.5570 assertions/s.

3 runs, 4 assertions, 0 failures, 0 errors, 0 skips
```

`terminal`

Yep, it passes and all is right with the world. If I had ideas of design patterns to use I would implement them now. But the key in this phase is to only change the code, not the tests. If you change the tests in this phase you are redesigning and not refactoring.

**NOTE**: Tests need to be written and rewritten. Far too often I see tests that were written once and never revisited again. If you to the TDD Cycle right, you will be revisiting and updateing your tests *more* than you change your code.

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
```

`terminal`

Again, we only let our Engineering mindset consider the test design after all code under test is properly refactored, while all the tests continue to pass cleanly. And by delaying our Engineering mindset until the *Refactoring* step we have given our Consumer mindset the ability to design the optimum API.


Conclusion
----------

You've now written tests to go from **Error** to **Failure** to **Passing** through TDD's **Red**, **Green, **Refactor** steps. The dirty little secret is that you will to *Red* *Green* *Red* *Green* *Red* *Green* and then *Refactor*. In this process the tests provide sufficent coverage for the API you design for your code's consumers. You then use the feedback your tests provide to make changes. The end result is you will have the confidence to make improvements to your software because you enjoy the assurance that you aren't breaking behavior.

The TDD Cycle also promotes looking at your code from difference mindsets. You look at your code from multiple perspectives, from the Consumer to the Engineer, from the Designer to the Maintainer, allowing you to entertain both your creativity and your focus/completeness/something (!!). The more you practice TDD, the better at these mindsets you get, and the better code you will write.
