Chapter 1 - Minitest API
========================

When you get right down to it, Minitest is pretty simple. If you are familiar with testing software it does what you would expect, and if you are new to testing software it is easy to learn thanks to its small surface area. That isn't to say Minitest isn't powerful, or full featured, just that it was designed to do it's job and to get out of the way. Writing tests with Minitest is easy. And, as it turns out, pretty fun.

<aside>
<p>This book assumes you have a Ruby 2.0 and Minitest 5 installed. Please follow the <a href="prerequisites">prerequisites</a> if you need to install these.</p>
</aside>

The best way to learn something is to use it, not read about it. So let's build something! We will use Minitest and explain its features as we go.

There are four main steps to writing tests using Minitest:

1. Create a test class
2. Create a test method
3. Call an assertion method
4. Create support methods

These steps are encompassed by the Minitest API, or Application Programming Interface. You use Minitest the way you would use any other software library. 

*NOTE*: When I was first learning how to test I was confused and intimidated by the API. I think those reading this may also be in the same boat. What do I say here to let them know I am on their side and we can learn this together?

Test Class
----------

Let's create a new file named `minibot.rb` to hold our code. In order to use Minitest we need to require it in our code. We also want to make sure we are using the gem version of Minitest and not the one that is distributed with Ruby. To do that we use the `gem` command.

```ruby
gem "minitest"
require "minitest/autorun"
``` `minibot.rb`

To create a test we need to create a new class that inherits from `Minitest::Test`. This will be our **Test Class**. Some projects, like [Ruby on Rails](http://rubyonrails.org/), will provide other test classes that provide project-specific functionality. The testing behavior of these classes is shared through inheritance. Your test classes don't need to inherit directly from `Minitest::Test`, but it needs to have `Minitest::Test` in its ancestry.

It is common for the test class name to describe the subject of the test. Let's create a new test class named `TestMinibot`.

```ruby
gem "minitest"
require "minitest/autorun"

class TestMinibot < Minitest::Test

end
``` `minibot.rb`

<aside>
<p>Earlier versions of Minitest used <code>MiniTest::Unit::TestCase</code> instead of <code>Minitest::Test</code>. They are functionally equivalent, but <code>MiniTest::Unit::TestCase</code> is deprecated. See [appendix C](appendix/api-changes) for a more detailed description of the changes between Minitest 5 and MiniTest 4.</p>
</aside>

This code looks pretty simple, but it actually does quite a bit. By inheriting from `Minitest::Test` our class has all the behavior it needs to run our tests. All we need to do is implement them.

Test Methods
------------

We implement individual tests by creating **Test Methods** within out **Test Class**. These methods are no different that any other method, except for how they are named. They must begin with `test_` for Minitest to recognize that they are test methods and treat them as such.

Let's create a test method. Like test classes, test methods should have a descriptive name. We don't know what a "minibot" is just yet, so the only requirement is that it starts with `test_`. How about `test_minibot`?

```ruby
gem "minitest"
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def test_minibot

  end
end
``` `minibot.rb`

Like our **Test Class**, the **Test Methods** are containers tests. So far we have been creating locations for the test, but haven't actually tested anything. To do this we will use one of the many **Assertion Methods** that Minitest provides.

Assertion Methods
-----------------

Minitest checks for correctness by asserting that behavior is correct. If an assertion fails then Minitest will log it as a failure. The base assertion is the `assert` method. It evaluates the value passed to it, and if the value is `nil` or `false` the assertion will fail. For now, let's create the most basic assertion we can.

```ruby
gem "minitest"
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def test_minibot
    assert true
  end
end
``` `minibot.rb`

In our Test Method we are calling the `assert` method, and passing it a boolean value. If the value is `true`, the test till pass. If it is `false`, the test will fail.

Minitest also provides several specialied assertion methods, such as `assert_equal`, `assert_in_delta`, and `assert_kind_of`. A full list of assertions with examples is found in [appendix A](appendix/assertions).

So let's run our test and see if it passes. To do this we will tell `ruby` to run our file from the terminal.

```shell
$ ruby minibot.rb
Run options: --seed 32365

.

Finished in 0.000784s, 1275.5102 runs/s, 1275.5102 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
``` `terminal`

<aside>
<p>We explain more robust ways to run tests in <a href="ch4-running-test">chapter 4</a>, but for now we'll run the tests this way.</p>
</aside>

Good job! The output tells us Minitest ran our assertion and there were no failures or errors. Try changing the value to `false` and see what output you get.

So far we have introduced a **Test Class**, created a **Test Method**, used an **Assertion Method** to specify our expectations. But there is one last component to the Minitest API: **Support Methods**.

*NOTE*: Probably should explain the "expected, actual" order in assertions somewhere. This seems like a good place...

Support Methods
---------------

Minitest provides methods to execute code before and after a *test method* is run. These methods are `setup` and `teardown`, respectively. They can be used for anything from initializing common collaborators in your tests to to creating and rolling back database transactions.

```ruby
def setup
  # I get called before every test method
end

def teardown
  # I get called after every test method
end
```

For our purpose, let's use an instance variable named `@minibot`. The `setup` method will be called before every test method, setting `@minibot` to `true` before each test. And the `teardown` method will be called after each test, setting `@minibot` to nil. The test methods can use `@minibot` in the assertions.

Let's 

```ruby
gem "minitest"
require "minitest/autorun"

class TestMinibot < Minitest::Test
  def setup
    @minibot = true
  end

  def test_minibot
    assert @minibot
  end

  def teardown
    @minibot = nil
  end
end
``` `minibot.rb`

One way to improve the readability of your tests is to move shared functionality out of the test methods and into new methods. Because Minitest is using normal Ruby classes you can use all the approaches you would use to remove duplication in your test classes as you would your application classes. But be careful, the purpose of the test code is to clearly describe the application's API and intent, and too much indirection may hurt readability. I generally prefer a little more duplication in my tests than I prefer in my application code.

Conclusion
----------

You've created your own **test class**, added a **test method**, and called an **assertion method** on an object created by the **support methods**. Simple, right? This is really all there is to it. Tests are just classes with methods that verify the behavior of your code with assertions.

You are now familiar with the Minitest API. The notion of API is important: it communicates how the software is expected to be used. Your tests will also communicate how your code is expected to be used by verifying how the API works.
