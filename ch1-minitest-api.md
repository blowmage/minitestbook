Minitest API
============

When you get right down to it, Minitest is pretty simple. If you are familiar with testing software it does what you would expect, and if you are new to testing software it is easy to learn thanks to its small surface area. That isn't to say Minitest isn't powerful, or full featured, just that it was designed to do it's job and to get out of the way. Writing tests with Minitest is easy. And, as it turns out, pretty fun.

There are four main steps to writing tests using Minitest:

1) Create a test class
2) Create a test method
3) Call an assertion method
4) Create support methods

These steps are encompassed by the Minitest API, or Application Programming Interface. This simply means that you use Minitest the way you would use any other software library. (If you need to start even earlier, please see appendix #1 for installing Ruby, and appendix #2 for installing Minitest.) So let's get started!

Hello World!
------------

Before we can write tests we need to have some code to test. It is appropriate to start with a simple "Hello World!" example. Because Ruby makes this incredibly easy let's instead start with a simple object that will generate the "Hello World!" greeting:

    class HelloWorld
      def hello
        "Hello World!"
      end
    end
{hello_world.rb}

Test Class
----------

We want to test our HelloWorld object using Minitest. What is the first step? To create a new class that inherits from `Minitest::Test`. This will be our **Test Class**, and it needs to have `Minitest::Test` in its ancestry.

    class HelloWorld
      def hello
        "Hello World!"
      end
    end

    class TestHelloWorld < Minitest::Test
    
    end
{hello_world.rb}

In order to use Minitest we need to require it in our code. We also want to make sure we are using the gem version of minitest and not the one that is distributed with Ruby. To do that we use the `gem` command to tell Ruby to use the gem before you require Minitest in your test files. See appendix #2 Installing Minitest for more information on getting the right version of Minitest installed.

    class HelloWorld
      def hello
        "Hello World!"
      end
    end

    gem     "minitest"
    require "minitest"

    class TestHelloWorld < Minitest::Test
    
    end
{hello_world.rb}

{sidebar}
Earlier versions of Minitest used `MiniTest::Unit::TestCase` instead of `Minitest::Test`. They are functionally equivalent, but `MiniTest::Unit::TestCase` is deprecated. See appendix #3 for a more detailed description of the changes between Minitest 5 and MiniTest 4.
{endsidebar}

This code looks pretty simple, but it actually does quite a bit. By inheriting from `Minitest::Test` our class has all the behavior it needs to run our tests. All we need to do is implement them.

Test Methods
------------

We implement individual tests by creating **Test Methods** within out **Test Class**. These methods are no different that any other method, except for how they are named. They must begin with `test_` for Minitest to execute them. We want to test HelloWorld's `hello` method, so let's create a `test_hello` test method in our test class:

    class HelloWorld
      def hello
        "Hello World!"
      end
    end

    gem     "minitest"
    require "minitest"

    class TestHelloWorld < Minitest::Test
      def test_hello

      end
    end
{hello_world.rb}

Like our **Test Class**, the **Test Methods** are containers tests. So far we have been creating locations for the test, but now we are ready to exercise our HelloWorld object and verify that it behaves as expected. To do this we will use one of the many **Assertion Methods** that Minitest provides.

Assertion Methods
-----------------

Minitest checks for correctness by asserting that behavior is correct. If an assertion fails then Minitest will log it as a failure. The base assertion is the `assert` method. It evaluates the value passed to it, and if the value is `nil` or `false` the assertion will fail. In our case, we want to compare what we _expect_ HelloWorld#hello to return with what it _actually_ returns:

    class HelloWorld
      def hello
        "Hello World!"
      end
    end

    gem     "minitest"
    require "minitest"

    class TestHelloWorld < Minitest::Test
      def test_hello
        assert("Hello World!" == HelloWorld.new.hello)
      end
    end
{hello_world.rb}

In our Test Method we are calling the `assert` method, and passing it a boolean value. If `HelloWorld.new.hello` returns the string "Hello World!" then the test will pass. If not, it will fail. (See Appendix #4 for a complete list of all the assertions that Minitest provides.)

So let's run our test and see if it passes. To run the tests we need to tell Ruby to load the code we've written. We will also pass the `-r minitest/autorun` option to Ruby so Minitest will know to run the tests. We will explain what this and more in Chapter 4: Running Tests, but for now let's run the following:

    $ ruby -r minitest/autorun hello_world.rb
    Run options: --seed 45022

    # Running:

    .

    Finished in 0.001243s, 804.5052 runs/s, 804.5052 assertions/s.

    1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
{terminal}

Good job! The output tells us Minitest ran our assertion and there were no failures or errors. We have introduced a **Test Class**, created a **Test Method**, and used an **Assertion Method**. But there is one last component to the Minitest API: **Support Methods**.

Support Methods
---------------

Minitest provides methods to execute code before and after a *test method* is run. These methods are `setup` and `teardown`, respectively. They can be used for anything from initializing common collaborators in your tests to to creating and rolling back database transactions.

For our purpose, let's create an instance of HelloWorld and store it in the `@hello_world` instance variable. The `setup` method will be called before every test method, reinitializing `@hello_world` before each test. And the `teardown` method will be called after each test, setting `@hello_world` to nil. The test methods are free to use `@hello_world` instead of initializing HelloWorld.

    class HelloWorld
      def hello
        "Hello World!"
      end
    end

    gem     "minitest"
    require "minitest"

    class TestHelloWorld < Minitest::Test
      def setup
        @hello_world = HelloWorld.new
      end

      def test_hello
        assert("Hello World!" == @hello_world.hello)
      end

      def teardown
        @hello_world = nil
      end
    end
{hello_world.rb}

One way to improve the readability of your tests is to move shared functionality out of the test methods and into new methods. Because Minitest is using normal Ruby classes you can use all the approaches you would use to remove duplication in your test classes as you would your application classes. But be careful, the purpose of the test code is to clearly describe the application's API and intent, and too much indirection may hurt readability. I generally prefer a little more duplication in my tests than I prefer in my application code.

Conclusion
----------

You've created your own **test class** to test the HelloWorld object, created a **test method** to verify HelloWorld#hello behaves as expected, called an **assertion method** to verify the greeting is correct, and used the **support methods** to remove some duplication in your tests.

You are now familiar with the Minitest API. The notion of API is important: it communicates how the software is expected to be used. Your tests will also communicate how your code is expected to be used by verifying how the API works.
