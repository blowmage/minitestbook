Minibot: Responses
==================

In chapter 3 we used TDD to move rapidly from Red to Green on the Minibot API for adding rules. Let's update our TODO by marking the "Add a rule" task complete. We can also add a couple more tasks for listing triggers and listing a trigger's responses. We already have coverage for listing triggers, so let's mark that task as complete as well.

```plain
Minibot
=======

A simple bot for storing and matching memes.

- [x] Match a message string to a rule
- [x] Add a rule
- [ ] Remove a rule
- [x] List triggers
- [ ] List responses for a trigger
```

`todo.md`

List responses for a trigger
----------------------------

Now the question is how do we allow users to list the responses for a trigger? Given the design on Minibot so far, it probably makes sense to implement a `Minibot#responses` method that takes `trigger` as an argument.

As we start to implement a test that uses this new method we notice a common data usage patten emerge in our tests. We can add the same three "zomg" rules and the one "wtf" rule as we have done in the other tests, and assert the responses behavior according to those rules.

```ruby
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
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 43364

....E.

Finished in 0.001100s, 5454.5455 runs/s, 11818.1818 assertions/s.

1) Error:
TestMinibot#test_responses:
NoMethodError: undefined method `responses' for #<Minibot:0x007f85a3e194e0>
test/test_minibot.rb:54:in `test_responses'

6 runs, 13 assertions, 0 failures, 1 errors, 0 skips
```

We are back in the error state. Let's add the missing `Minibot#responses` method and see if we can into the failure state. This should start to feel very familiar to you by now. Remember how foreign this felt before? Now it feels natural and comforting. The test output is telling us what we should do next.

```ruby
def responses trigger
  # TODO: Return responses
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 46312

# Running:

.....E

Finished in 0.000962s, 6237.0062 runs/s, 13513.5135 assertions/s.

1) Error:
TestMinibot#test_responses:
NoMethodError: undefined method `size' for nil:NilClass
test/test_minibot.rb:55:in `test_responses'

6 runs, 13 assertions, 0 failures, 1 errors, 0 skips
```

Still in the error state. Let's return an object from `Minibot#responses` that has the method `size`. We can use an empty array like we have done for this error before.

```ruby
def responses trigger
  # TODO: Return responses
  []
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 14687

# Running:

...F..

Finished in 0.027992s, 214.3470 runs/s, 500.1429 assertions/s.

1) Failure:
TestMinibot#test_responses [test/test_minibot.rb:55]:
Expected: 3
Actual: 0

6 runs, 14 assertions, 1 failures, 0 errors, 0 skips
```

Back into the failure state. Now we can start working on the behavior. We need to return a list of 3 responses when given the trigger "zomg", and a list of one response when given the trigger "wtf". The correct way to do this is to collect the responses as they are added, but that would require changing code in both `Minibot#add` and `Minibot#responses`. My Worst Programmer mindset thinks there is a chance I could stumble on that implementation. It is probably faster to hardcode the data in the `responses` method based on the `trigger` argument.

```ruby
def responses trigger
  if trigger == "zomg"
    [ "http://i.imgur.com/49ORL0o.gif",
      "http://i.imgur.com/KqCfQPR.gif",
      "http://i.imgur.com/fc5LmfO.gif" ]
  elsif trigger == "wtf"
    [ "http://i.imgur.com/ULQl7.gif" ]
  end
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 59221

......

Finished in 0.001000s, 6000.0000 runs/s, 23000.0000 assertions/s.

6 runs, 23 assertions, 0 failures, 0 errors, 0 skips
```

From error to failure to passing. We can check the "List responses for a trigger" task as complete. The TDD Cycle we are following is working for us and we are moving faster and faster.

Random Responses
----------------

If we take a look at our tests, we have some good coverage for adding rules, listing triggers, and listing responses. But we don't have great coverage for matching responses. For instance, we don't specify how Minibot should behave when there are multiple responses on a matched trigger. We want Minibot to pick a random response, so let's write a test for that.

Testing random behavior is not as easy as you might think. There are always anomalies to how random Ruby's `rand` can be. If we matched the "zomg" trigger 300 times, we would expect to match all three triggers ~100 times. But in reality the actual count can vary wildly. This is where another specialized assertion method, `assert_in_delta`,  comes in handy.

The `assert_in_delta` method takes 3 arguments, the expected value, the actual value, and the delta that the values can vary while still passing the assertion. The default delta is 0.001, but we will need this to be larger, much larger.

We will run the match 300 times, and expect that all three responses are returned 100 times. But we will set the delta to be 33. So if each responses total is between 67 and 133 the test will pass.

```ruby
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
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 7881

# Running:

....F..

Finished in 0.001241s, 5640.6124 runs/s, 19339.2425 assertions/s.

1) Failure:
TestMinibot#test_match_returns_random_response [test/test_minibot.rb:21]:
Expected |100 - 300| (200) to be <= 33.

7 runs, 24 assertions, 1 failures, 0 errors, 0 skips
```

The `test_match_returns_random_response` test fails because we are only returning one response. We need to make `Minibot#match` return a random response instead.

```ruby
def match message
  # TODO: Refactor. Response is coupled to test.
  if message == "ZOMG! This is cool!"
    responses("zomg").sample
  end
end
```

But now the `test_match` test fails most of the time:

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 34178

....F..

Finished in 0.027894s, 250.9500 runs/s, 932.1001 assertions/s.

  1) Failure:
TestMinibot#test_match [test/test_minibot.rb:9]:
--- expected
+++ actual
@@ -1 +1 @@
-"http://i.imgur.com/49ORL0o.gif"
+"http://i.imgur.com/KqCfQPR.gif"

7 runs, 26 assertions, 1 failures, 0 errors, 0 skips
```

Although occasionally, the test will pass:

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 33754

.......

Finished in 0.001362s, 5139.5007 runs/s, 19089.5742 assertions/s.

7 runs, 26 assertions, 0 failures, 0 errors, 0 skips
```

The `test_match` test passes occasionally because that test is only registering one rule, and is expecting only that response. It is possible change the test to make it aware of all three responses, but violates the purpose of these tests. What quick change can we make to get this test passing again?


We can make this test pass by capturing the responses in `Minibot#add` the same way we capture the triggers. This certainly seems like the eventual solution, but again involves changes in multiple methods. This seems to be too big of a change to make during a failing test. We want to get to Green as fast as possible, and we know that if there are more than one "zomg" in `@trigger` then we can use a random response. Otherwise we must use the hardcoded URL from before. So let's add a call to `@triggers.count("zomg")` in `Minibot#match` to determine if we return a random value or the hardcoded one.

```ruby
def match message
  # TODO: Refactor. Response is coupled to test.
  if message == "ZOMG! This is cool!"
    # TODO: Remove @trigger count
    if @triggers.count("zomg") > 1
      responses("zomg").sample
    else
      "http://i.imgur.com/49ORL0o.gif"
    end
  end
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 57173

# Running:

.......

Finished in 0.001329s, 5267.1181 runs/s, 19563.5816 assertions/s.

7 runs, 26 assertions, 0 failures, 0 errors, 0 skips
```

This is the worst code added so far. Nobody should feel good about this code. That pride you feel for finding a quick and elegant way to get that test passing? Squash it. That is the Worst Programmer mindset. The only good news here is that all the tests are passing, and that we now have enough test coverage to give us confidence that we can change the implementation without breaking the API. And take heart in knowing that this code will not survive the final step of the TDD Cycle: Refactoring.

Recap
-----

The Customer mindset wants to ensure that the API works in the "Make it Red" step, but the Worst Programmer wants to ensure that we spend as little time being *Red* as possible, taking any and every shortcut to get back to *Green*. The result is a very rapid feedback cycle on the design if the API that you are providing at the cost of your code quality.

It should feel like you are trolling yourself. The Consumer mindset exercises your creativity by finding the edges of what is acceptable behavior for your API, while the Worst Programmer mindset exercises your critical thinking by taking every short cut possible to keep the code passing the tests. The result is rapid feedback on the design of the API, and a suite of tests that check the API implementation for correctness. The focus is all on API design and test coverage. The tests enforce the design the API. Now we are ready to move on to the final phase of the TDD Cycle.
