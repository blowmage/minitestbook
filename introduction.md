Introduction
============

Why Minitest?
-------------

**NOTE**: Here is where we sell the reader on testing and what benefits testing has.

Is your code hard to maintain? Does it take longer to add a feature the older your code is? This is a common situation, and testing is a wonderful way to write better code that is easier to change and you can have more confidence in.

**NOTE**: Explain what Minitest's philosophy is and what value it will give the user.

Minitest is more than a testing library. It embodies a specific philosophy for testing code. It also embodies a larger philosophy of how software should be designed. Unix, TDD, XP

Minitest embodies these philosophies, but that doesn't mean you can't use Minitest if you have a different view. Minitest is intended to be used, and so it doesn't 

Getting Started
---------------

Project Layout
--------------

In order to use Minitest we need to require the library in our code. We also want to make sure we are using the gem version of Minitest and not the one that is distributed with Ruby. To do that we use the `gem` command.

```ruby
gem "minitest"
require "minitest/autorun"
```

`test/helper.rb`


Running Tests
-------------

Building Minibot
----------------

I love bots, all kids of bots. IRC bots. Campfire bots. Twitter bots. They make smile (for the most part). I want to build a bot that is well designed, and I'm going to use Minitest to help me get there.

Since I don't want to get lots in the weeds of all the things that a bot can do, let's start simple. Here is a TODO list of the things I want this bot to do. We will work off this list and add to it as we progress.

```plain
Minibot
=======

A simple bot for storing and matching memes.

- [ ] Add a new matching rule
- [ ] List all matching rules
- [ ] Remove a matching rule
- [ ] Match an given string to an existing rule
```

`todo.md`

1. Directly through the command line
2. Rake#testtask
3. m gem
4. autotest







```ruby
gem "minitest"
require "minitest/autorun"

class TestMinitest < Minitest::Test
  def test_minitest
    assert "Minitest is pretty great!"
  end
end
```

`test_minitest.rb`

```shell
$ ruby test_minitest.rb
Run options: --seed 32365

.

Finished in 0.000784s, 1275.5102 runs/s, 1275.5102 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

`terminal`





So far we have been passing `-r minitest/autorun` to ruby when we want to run the tests in a file. But what if we want to run all the tests in our application? Or what if we want to run just one test method in a file?

Minitest provides an object called a Runner that will run the tests for us and print the output. This makes running tests very easy. All we need to do is to load the runner with our code. Let's do this by passing `-r minitest/autorun` to Ruby to tell it to load the runner as we load our code:

