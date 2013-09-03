TDD Cycle: Refactor
===================

So far we have been racing between Red and Green, driving our code from error to failure to passing and back again. We've been focusing on the API we are exposing to the consumers of our code, and intentionally not focusing on the quality of that code. We've been supressing the desire to make the code right, but that is finally about to change.

Step 3: Make it Right
---------------------

Now that the Customer and Worst Programmer mindsets have gone rounds we have a suite of tests that demonstrate the API and show how it is intended to work. Now we can focus on changing the code so that it is correct. This step is also known as the *Refactoring* step, because we want to be able to make changes to how our code is implemented without changing the API we have defined. Let's switch from the Consumer and Worst Programmer mindsets to the Engineer mindset and start thinking about what improvements we should make to the code. Because of our hard work so far we have some good test coverage so that we will know if we change the expected behavior the next time we run the tests.

Let's take a look at the state of our code up to this point:

```ruby
class Minibot
  def add trigger, response
    @triggers ||= []
    @triggers << trigger
  end

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

  def triggers
    @triggers.uniq
  end

  def responses trigger
    if trigger == "zomg"
      [ "http://i.imgur.com/49ORL0o.gif",
        "http://i.imgur.com/KqCfQPR.gif",
        "http://i.imgur.com/fc5LmfO.gif" ]
    elsif trigger == "wtf"
      [ "http://i.imgur.com/ULQl7.gif" ]
    end
  end
end
```

`lib/minibot.rb`

As I look at the state of our code, the first thing I want to change is to remove the `@triggers` list and the hardcoded responses. My Engineer mindset doesn't like that. How can we change the implementation without changing the API? We should collect the rules as they are added. I think a Hash with a default Array value would be perfect for this. It will return an empty array if the key does not exist in the array, and you can add values to a key.

```ruby
rules = Hash.new { |hash, key| hash[key] = [] }

rules["zomg"] #=> []
rules["zomg"] << "response 1"
rules["zomg"] << "response 2"
rules["zomg"] #=> ["response 1", "response 2"]

rules["wtf"] #=> []
rules["wtf"] << "response A"
rules["wtf"] #=> ["response A"]
```

Because the tests are passing, and we have some pretty good coverage of the API we are providing, I'm comfortable making some significant changes to `lib/minibot.rb` to use this data structure. As long as the tests continue to pass. I think we should instantiate this hash in Minibot's `initialize` method, and use it to store triggers and responses.

```ruby
class Minibot
  def initialize
    @rules = Hash.new { |hash, key| hash[key] = [] }
  end

  def add trigger, response
    @rules[trigger] << response
  end

  def match message
    # TODO: Refactor. Response is coupled to test.
    if message == "ZOMG! This is cool!"
      @rules["zomg"].sample
    end
  end

  def triggers
    @rules.keys
  end

  def responses trigger
    @rules[trigger]
  end
end
```

`lib/minibot.rb`

Much better! Does it continue to pass?

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 34829

# Running:

.......

Finished in 0.001313s, 5331.3024 runs/s, 19801.9802 assertions/s.

7 runs, 26 assertions, 0 failures, 0 errors, 0 skips
```

`terminal`

Yep, it passes and my confidence is growing. I feel much better about the quality of the code. It's not perfect, as we are still coupled to the test data for matching a rule. We really should make that better as well. I'm surprised we got this far by faking the matching of a message to a rule.

I think the easiest way to match a trigger to a message is to use the `String#include?` method. The only wrinkle with this is that it is case sensitive. The API we are exposing must matching triggers to messages in a case insensitive way. For now I would downcase both the message and the trigger on the check.

```ruby
trigger = "zomg"
message = "ZOMG! This is cool!"

message.include? trigger #=> false
message.downcase.include? trigger.downcase #=> true
```

To match the message to the rules, let's iterate through the rules hash and if the trigger matches the message, return a random response. Here is what that could look like:

```ruby
def match message
  @rules.each do |trigger, responses|
    if message.downcase.include? trigger.downcase
      return responses.sample
    end
  end
  nil
end
```

I am starting to love this code. Its clear and obvious, and I'm very confident in it right now. Does it still pass?

```shell
$ ruby -Ilib:test test/test_minibot.rb

Run options: --seed 9405

# Running:

.......

Finished in 0.001610s, 4347.8261 runs/s, 16149.0683 assertions/s.

7 runs, 26 assertions, 0 failures, 0 errors, 0 skips
```

If I had additional ideas for design patterns or other implementation details I would implement them now. But the key in this phase is to only change the code, not the tests. If you change the tests in this phase you are redesigning and not refactoring.

Recap
-----

You've now written tests to go from **Error** to **Failure** to **Passing** through TDD's **Red**, **Green, **Refactor** steps. The dirty little secret is that you will to *Red* *Green* *Red* *Green* *Red* *Green* and then *Refactor*. In this process the tests provide sufficent coverage for the API you design for your code's consumers. You then use the feedback your tests provide to make changes. The end result is you will have the confidence to make improvements to your software because you enjoy the assurance that you aren't breaking behavior.

The TDD Cycle also promotes looking at your code from difference mindsets. You look at your code from multiple perspectives, from the Consumer to the Engineer, from the Designer to the Maintainer, allowing you to entertain both your creativity and your focus/completeness/something (**!!**). The more you practice TDD, the better at these mindsets you get, and the better code you will write.
