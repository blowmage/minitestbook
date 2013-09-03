TDD Cycle: Red and Green
========================

I've often been asked, "How is writing code with tests any different than just writing code?". This is one of my favorite questions to answer, and by the end of this chapter you will be able to answer it too.

Now that we are familiar with the Minitest API, let's take a look at how we can use Minitest to help us design our code. We will be using a practice called Test Driven Development (TDD) and the TDD Cycle. TDD is not intended to be used for quality assurance of our software; TDD is intended to be used to help us drive the design of our code. This means we want to write our tests before we write our code, and use the tests to inform what code we write.

If this doesn't make sense to you, don't worry, we will step through the TDD Cycle several times before we're done.

Step 1: Make it Red
-------------------

In chapter 1 we adding a new *test method* for the first TODO: "Match a message string to a rule". When writing this new test method we had a decision to make: What do we want the Minibot API to be for adding a rule, matching a message, and returning the response? What information do we need to provide to add the rule? How do we know that a rule was added? We need to put ourselves in the mindset of those that consume our library: how would we want to use this code? This process is key: by consuming the code first we get to change our mindset and consider what the most appropriate API should be. We are not concerned with implementation details, we are performing a *design* exercise.

Here is what our test file looks like now:

```ruby
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def test_match
    minibot = Minibot.new
    minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    response = minibot.match "ZOMG! This is cool!"
    assert_equal "http://i.imgur.com/49ORL0o.gif", response
  end
end
```

`test/test_minibot.rb`

So far the only code we have written in our *test class*, so we expect our tests to fail. To run the test we pass the file to ruby:

```shell
$ ruby test/test_minibot.rb
Run options: --seed 44520

# Running:

E

Finished in 0.000739s, 1353.1800 runs/s, 0.0000 assertions/s.

1) Error:
TestMinibot#test_match:
NameError: uninitialized constant TestMinibot::Minibot
test/test_minibot.rb:5:in `test_match'

1 runs, 0 assertions, 0 failures, 1 errors, 0 skips
```

`terminal`

Interestingly, instead of generating a failure it generated an error. The error occurred because our new test method is trying to create an new object from a class that does not exist. This is okay, Minitest expects errors to happen and will display the appropriate information.

Handling errors is part of our design process. We want to wrote tests to use our code before we write our code. And we want to make this test pass by writing as little code as possible. Let's start by fixing the 'uninitialized constant TestMinibot::Minibot' error by adding a class definition for Minibot in our 'lib/minibot.rb` file:

```ruby
class Minibot

end
```

`lib/minibot.rb`

And requiring the class definition in our test file after the Miniitest require:

```ruby
require "minitest/autorun"
require "minibot"
```

`test/test_minibot.rb`

That should take care of the missing constant error. To run this test now we also need to tell Ruby to load the lib directory as well as the test directory. We do this with the `-Ilib:test` option:

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 8675

# Running:

E

Finished in 0.000790s, 1265.8228 runs/s, 0.0000 assertions/s.

1) Error:
TestMinibot#test_match:
NoMethodError: undefined method `add' for #<Minibot:0x007f90f6df89f8>
test/test_minibot.rb:7:in `test_match'

1 runs, 0 assertions, 0 failures, 1 errors, 0 skips
```

Now we have a different error, this time because the test is calling a method that does not exist. The Minibot constant exists, but the `Minibot#add` method does not. Let's create the `add` the method, but not the implementation, and rerun the test:

```ruby
class Minibot
  def add trigger, response
    # TODO: Save this data
  end
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb
Run options: --seed 4935

E

Finished in 0.000866s, 1154.7344 runs/s, 0.0000 assertions/s.

1) Error:
TestMinibot#test_match:
NoMethodError: undefined method `match' for #<Minibot:0x007fadc3eb87f0>
test/test_minibot.rb:8:in `test_match'

1 runs, 0 assertions, 0 failures, 1 errors, 0 skips
```

Another undefined method, this time for `Minibot#match`. Our test is asking the Minibot object to match a message and return a response, but that method has not yet been implemented and Minitest is telling us this. Let's add an empty method to clear this error:

```ruby
class Minibot
  def add trigger, response
    # TODO: Save this data
  end

  def match message
    # TODO: Match message to triggers
  end
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb
Run options: --seed 43768

# Running:

F

Finished in 0.033050s, 30.2572 runs/s, 30.2572 assertions/s.

1) Failure:
TestMinibot#test_match [test/test_minibot.rb:9]:
--- expected
+++ actual
@@ -1 +1 @@
-"http://i.imgur.com/49ORL0o.gif"
+nil

1 runs, 1 assertions, 1 failures, 0 errors, 0 skips
```

Ahh, the output is slightly different. Instead of displaying an "Error", Minitest is now displaying a "Failure". This is considered *Red*. Before we had *errors*; classes or methods that were called by did not exist. Now we have a *failure*, the code's behavior does not meet the test's expectations. In this case, the test expected a URL value to be returned, but got nil instead. We want our tests to drive our code from error to failure to pass. Let's fix this failure and get our test passing.

Now that we are in a failure state, we can focus on moving into the passing state. Like the previous step, we want to move from failure to pass as quickly as possible. We don't want to get bogged down in implementation details.

When we were writing our tests we put ourselves in the mindset of the consumers of our code. We thought through what API we wanted to use, not just what was easy to expose. Now we are changing the code to fulfill the expectations we have placed on the API, and we need change mindset once again. instead of focusing on implementation and correctness of our code, we want to focus on how quickly we can get the tests to pass and return to the customer mindset. To do this we need to put on the personality of the Worst Programmer in the World. We want to take any and all shortcuts needed to get the test to pass, even if this means intentionally writting bad or buggy code that won't work outside of our tests.

Step 2: Make it Green
---------------------

How would the Worst Programmer in the World make this test pass? Probably hard code the implementation so it matches the expectations of the test. Let's do this by having the `match` method return the response string that the test is expecting.

```ruby
class Minibot
  def add trigger, response
    # TODO: Save this data
  end

  def match message
    # TODO: Refactor. Response is coupled to test.
    "http://i.imgur.com/49ORL0o.gif"
  end
end
```

`lib/minibot.rb`

What an awful solution! Surely Minitest wouldn't allow something as bad as that to pass, right?

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 8245

.

Finished in 0.000786s, 1272.2646 runs/s, 1272.2646 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

`terminal`

It passes! Our terrible code passes! Passing is considered *Green*. But the code feels dirty, doesn't it? The `add` method doesn't do anything and the `match` method is hardcoded to the test expectation. Surely this isn't how we want our code to be.

### No Match

Time for another mindset change. We want to return to our Consumer Mindset and write more tests that flesh out more of the expected behavior. What if the message doesn't match the added rule? What are our expectations for `Minibot#add`? Can we add multiple rules? Can we add multiple rules on the same trigger? Let's find out by writing some more tests. First, let's add another test for what should happen when the message doesn't match a rule:

```ruby
def test_no_match
  minibot = Minibot.new
  minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
  response = minibot.match "ZOINKS! This doesn't match!"
  assert_nil response
end
```

We've decided that Minibot will return nil if the message doesn't match a rule. Running the tests now puts us back into *Red*.

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 2222

# Running:

.F

Finished in 0.001220s, 1639.3443 runs/s, 1639.3443 assertions/s.

1) Failure:
TestMinibot#test_no_match [test/test_minibot.rb:16]:
Expected "http://i.imgur.com/49ORL0o.gif" to be nil.

2 runs, 2 assertions, 1 failures, 0 errors, 0 skips
```

The `test_match` test passes, but `test_no_match` doesn't. We want to get back to *Green* quickly. How can we do it? Let's change mindset back to the Worst Programmer and find the most expedient way possible, sacrificing correctness for **???** In this case, we can have `Minibot#match` check for message and return the expected URL only when the message is the one from `test_match`. Let's try that to get this test passing:

```ruby
def match message
  # TODO: Refactor. Response is coupled to test.
  "http://i.imgur.com/49ORL0o.gif" if message == "ZOMG! This is cool!"
end
```

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 9954

# Running:

..

Finished in 0.000816s, 2450.9804 runs/s, 2450.9804 assertions/s.

2 runs, 2 assertions, 0 failures, 0 errors, 0 skips
```

With this change both the `test_match` and `test_no_match` tests pass! We let the tests drive our code implementation and we are *Green* again. We are starting to describe a usable API for Minibot, but I have no confidence in this code because it is so specific to the tests.

Recap
-----

The first step in TDD is to get your tests into a Red state. Tests are written with the Customer in mind, meaning they are focused on the API design and not the implementation. Any errors in your tests need to be turned into failures as quickly as possible. You want to do as little coding as possible to move from an error state to a failure state.

The second step in TDD is to get your tests into a Green state as quickly as possible. To do that use the Worse Programmer mindset and take any and all shortcuts necessary to get the tests passing. Think of the Red state as a game, you want to get out of Red to Green as quickly as you can. But once you are Green you want to return to Red as quickly as you can.

