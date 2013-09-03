Write a Test
============

When you get right down to it, Minitest is pretty simple. If you are familiar with testing software it does what you would expect, and if you are new to testing software it is easy to learn thanks to its small surface area. That isn't to say Minitest isn't powerful, or full featured, just that it was designed to do it's job and to get out of the way. Writing tests with Minitest is easy. And, as it turns out, pretty fun.

There are four main steps to writing tests using Minitest:

1. Create a test class
2. Create a test method
3. Call an assertion method
4. Create support methods

These steps are encompassed by the **Minitest API**, or Application Programming Interface. You use Minitest the way you would use any other software library. The best way to learn something is to use it, not read about it. So let's build something! We will use Minitest and explain its features as we go.

Building Minibot
----------------

I love bots, all kids of bots. IRC bots. Campfire bots. Twitter bots. They make smile (for the most part). So let's use Minitest to help us build a bot! We don't want to get lost in the weeds of all the things that a bot can do, let's start simple. Here is a TODO list of bot features. Let's will work off this list and add to it as we progress.

```plain
Minibot
=======

A simple bot for storing and matching memes.

- [ ] Match a message string to a rule
```

`todo.md`

To start with we will match a message to one of the bot's rules. For example, a messages that includes the word "zomg" in it would get an animated gif in response. If you sent the message "ZOMG! This is cool!", the bot would respond "http://i.imgur.com/49ORL0o.gif".

Right away we know we will have the following pieces of data:

<table>
  <tr>
    <td><strong>trigger</strong></td><td>"zomg"</td>
  </tr>
  <tr>
    <td><strong>response</strong></td><td>"http://i.imgur.com/49ORL0o.gif"</td>
  </tr>
  <tr>
    <td><strong>message</strong></td><td>"ZOMG! This is cool!"</td>
  </tr>
</table>

We are going to write a test *before* we write the code. This is known as **test-first**. If we write the code first and the test after it would be considered **test-after**. Test Driven Development is a test-first approach, while Quality Assurance is typically a test-after approach. We will be using test-first and write tests before code.

Test Class
----------

Let's create a new file in our `test` directory named `test/test_minibot.rb` to hold our test code. To create a test we need to create a new class that inherits from `Minitest::Test`. This will be our **Test Class**. Some projects, like [Ruby on Rails](http://rubyonrails.org/), may provide other test classes with project-specific functionality. The testing behavior of your test class is provided by inheritance. Your test classes don't need to inherit directly from `Minitest::Test`, but it needs to have `Minitest::Test` in its ancestry.

It is common for the test class name to describe the subject of the test. Let's create a new test class named `TestMinibot`.

```ruby
require "minitest/autorun"

class TestMinibot < Minitest::Test

end
```

`test/test_minibot.rb`

<aside>
<p>Earlier versions of Minitest used <code>MiniTest::Unit::TestCase</code> instead of <code>Minitest::Test</code>. They are functionally equivalent, but <code>MiniTest::Unit::TestCase</code> is deprecated. See [appendix C](appendix/api-changes) for a more detailed description of the changes between Minitest 5 and MiniTest 4.</p>
</aside>

This code looks pretty simple, but it actually does quite a bit. By inheriting from `Minitest::Test` our class has all the behavior it needs to run our tests. All we need to do is implement them.

Test Methods
------------

We implement individual tests by creating **Test Methods** within out **Test Class**. These methods are no different that any other method, except for how they are named. They must begin with `test_` for Minitest to recognize that they are test methods and treat them as such.

Let's create a test method. Like test classes, test methods should have a descriptive name. We don't know what a "minibot" is just yet, so the only requirement is that it starts with `test_`. How about `test_minibot`?

```ruby
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def test_match

  end
end
```

`test/test_minibot.rb`

Like our **Test Class**, the **Test Methods** are containers tests. So far we have been creating locations for the test, but haven't actually tested anything. One of the common approaches to a test method is [Arrange Act Assert](http://c2.com/cgi/wiki?ArrangeActAssert).

### Arrange Act Assert

A test method is intended to perform a unit of work, and verify that the code behaves as expected. One way to organize this is to perform the following steps within the test method:

1. **Arrange** the necessary inputs and preconditions.
2. **Act** the intended behavior.
3. **Assert** the results are as expected.

What does this mean for our "Match a message string to a rule" task? We need to arrange a rule in Minibot to be matched, act on Minibot to match the message with the rule, and assert that the response to the message is what we expect. Because we are doing test-first we get to decide what Minibot's API will be as we write this test. So we can write the code that we want to use.

Here is what the *arrange* step may look like, creating a new Minibot object and registering a rule for "zomg" trigger and response:

```ruby
minibot = Minibot.new
minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
```

And here is what the *act* step may look like, matching the message and returning the response:

```ruby
response = minibot.match "ZOMG! This is cool!"
```

To perform the *assert* step we need to use one of the many **Assertion Methods** that Minitest provides.

Assertion Methods
-----------------

Minitest checks for correctness by asserting that behavior is correct. If an assertion fails then Minitest will log it as a failure. The base assertion is the `assert` method. It evaluates the value passed to it, and if the value is `nil` or `false` the assertion will fail. This is what the most basic assertion would look like:

```ruby
assert(response == "http://i.imgur.com/49ORL0o.gif")
```

In this code we are calling the `assert` method and passing it a boolean value. If the value is `true` the test will pass. If it is `false` the test will fail. We can also provide an optional second argument to describe the circumstances should the assertion fail:

```ruby
assert(response == "http://i.imgur.com/49ORL0o.gif",
       "Expected #{response} to equal http://i.imgur.com/49ORL0o.gif")
```

As you might expect, a good deal of value is found in the the error message. So much so that there are special-case assertion methods that generate this message for you. The assertion method `assert_equal` provides the logic and error message for checking the equality of two variables:

```ruby
assert_equal "http://i.imgur.com/49ORL0o.gif", response
```

<aside>
  <h3>Assertion argument order</h3>
  <p>The order of arguments in most specialized assertions is "expected, actual", which is opposite of what you would use for variable assignment. The first argument is the value you expect, and the second is the value created earlier in the test method. This order can be the source of a good deal of confusion when learning the Minitest API. But don't get too frustrated, In my experience this order is internalized quicker than most expect.</p>
</aside>

Minitest provides several additional specialied assertion methods, such as `assert_includes`, `assert_in_delta`, and `assert_kind_of`. A full list of assertions with examples is found in [appendix A](appendix/assertions).

Now that we have our test method and assertion, this is what our test file looks like:

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

So far we have introduced a **Test Class**, created a **Test Method**, used an **Assertion Method** to specify our expectations. But there is one last component to the Minitest API: **Support Methods**.

Support Methods
---------------

Minitest provides hook methods to execute code before and after a *test method* is run. These methods are `setup` and `teardown`, respectively. They can be used for anything from initializing common collaborators in your tests to to creating and rolling back database transactions.

```ruby
def setup
  # I get called before every test method
end

def teardown
  # I get called after every test method
end
```

For our purpose, we can create an instance variable named `@minibot`. The `setup` method will be called before every test method, setting `@minibot` to `Minibot.new` before each test. And the `teardown` method will be called after each test, setting `@minibot` to nil. The test methods can use `@minibot` and call assertions on it.

This is an example of what our test file could look like using the `setup` and `teardown` support methods:

```ruby
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def setup
    @minibot = Minibot.new
  end

  def test_match
    @minibot.add "zomg", "http://i.imgur.com/49ORL0o.gif"
    response = @minibot.match "ZOMG! This is cool!"
    assert_equal "http://i.imgur.com/49ORL0o.gif", response
  end

  def teardown
    @minibot = nil
  end
end
```

`test/test_minibot.rb`

The `teardown` method here is to illustrate how it can be used, but it isn't really needed for this test code to work as the value is reset in `setup`. One use for the `teardown` method would be to perform a rollback on a database transaction created in `setup`.

One way to improve the readability of your tests is to move shared functionality out of the test methods and into new methods. Because Minitest is using normal Ruby classes you can use all the approaches you would use to remove duplication in your test classes as you would your application classes. But be careful, the purpose of the test code is to clearly describe the application's API and intent, and too much indirection may hurt readability. I generally prefer a little more duplication in my tests than I prefer in my application code. I will remove duplication in my code after repeating the same logic 2-3 times, but I won't remove duplication in my tests until I've repeated the same logic 5-7 times.

Recap
-----

You've created your own **test class**, added a **test method**, and called an **assertion method** on an object created by the **support methods**. Simple, right? This is really all there is to it. Tests are just classes with methods that verify the behavior of your code with assertions.

You are now familiar with the Minitest API. The notion of API is important: it communicates how the software is expected to be used. Your tests will also communicate how your code is expected to be used by verifying how the API works.
