Chapter 2 - TDD Cycle
=====================

Now that we are familiar with the Minitest API, let's take a look at how we can use Minitest to help us design our code. We will be using a practice called Test Driven Development (TDD) and the TDD Cycle. TDD is not intended to be used for quality assurance of our software; TDD is intended to be used to help us drive the design of our code. This means we want to write our tests before we write our code, and use the tests to inform what code we write.

If this doesn't make sense to you, don't worry, we will step through the TDD Cycle several times before we're done.

New Feature
-----------

Let's add a small feature to the code from chapter 1. We have a HelloWorld object which will give us a "Hello World!" greeting, but we want to customize the greeting for an individual. We would really like to make a greeting for Matz, the creator of Ruby, by making it say "Hello Matz!".

This is a simple feature, and one we could probably make without extensive test coverage. After all, the risk of our implementation being wrong is relatively small. However, we aren't using TDD to ensure the _correctness_ of our code as much as we are to inform the _design_ of our code. So to that end we will be taking what may seem like unnecessary steps, but hopefully by the end you will recognize how helpful they are.

Make it Red
-----------

Let's start by adding a new *test method* that will say hello to Matz. When writing this new test method we have a decision to make: What do we want the API to be for customizing the message? We need to put ourselves in the mindset of those that consume our library: how do we want to use this code? Do we create a HelloWorld instance specific to the receiver? (`HelloWorld.for("Matz").hello`) Do we have a different `hello` method that accepts the customization? (`HelloWorld.new.hello_for("Matz")`) Or do we pass the name we want to use to the existing `hello` method? (`HelloWorld.new.hello("Matz")`)

Let's decide on passing a `name` argument to HelloWorld#hello. We'll create a new method named `test_hello_to_matz` and pass "Matz" as an argument.

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

      def test_hello_to_matz
        assert("Hello Matz!" == HelloWorld.new.hello("Matz"))
      end

      def teardown
        @hello_world = nil
      end
    end
{hello_world.rb}

So far we have only changed our *test class*, so we expect our tests to fail:

    $ ruby -r minitest/autorun hello_world.rb

    # Running:

    .E

    Finished in 0.001275s, 1568.6275 runs/s, 784.3137 assertions/s.

      1) Error:
    TestHelloWorld#test_hello_to_matz:
    ArgumentError: wrong number of arguments (1 for 0)
        hello_world.rb:2:in `hello'
        hello_world.rb:20:in `test_hello_to_matz'

    2 runs, 1 assertions, 0 failures, 1 errors, 0 skips
{terminal}

Interestingly, instead of generating a failure it generated an error. The error occurred because our new test method passed an argument to the `hello` method when it didn't expect one. This is okay, Minitest expects errors to happen and will display the appropriate information.

Let's fix the error by giving HelloWorld#hello an argument:

    class HelloWorld
      def hello(name)
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

      def test_hello_to_matz
        assert("Hello Matz!" == HelloWorld.new.hello("Matz"))
      end

      def teardown
        @hello_world = nil
      end
    end
{hello_world.rb}

Okay, let's rerun the tests and see if that helps:

    $ ruby -r minitest/autorun hello_world.rb
    Run options: --seed 17455

    # Running:

    EF

    Finished in 0.001197s, 1670.8438 runs/s, 835.4219 assertions/s.

      1) Error:
    TestHelloWorld#test_hello:
    ArgumentError: wrong number of arguments (0 for 1)
        hello_world.rb:2:in `hello'
        hello_world.rb:16:in `test_hello'


      2) Failure:
    TestHelloWorld#test_hello_to_matz [hello_world.rb:20]:
    Failed assertion, no message given.

    2 runs, 1 assertions, 1 failures, 1 errors, 0 skips
{terminal}

Now we have a failure and an error. We seem to be moving backwards, but we aren't really. Let's look at the two messages. The first error is from our original test that was previously passing. We added a parameter to `HelloWorld#hello` which changed the API that we were using. We have existing calls to `hello` that aren't providing a value, so we have broken our existing API. We need an API that allows for customizing the greeting as well as not customizing the greeting. We can do this by providing a default value along with the name parameter. Let's set the default value to nil in the method definition to see if we can get our original test to pass:

    class HelloWorld
      def hello(name = nil)
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

      def test_hello_to_matz
        assert("Hello Matz!" == @hello_world.hello("Matz"))
      end

      def teardown
        @hello_world = nil
      end
    end
{hello_world.rb}

Now when we run the tests we see that the first test method is passing again:

    $ ruby -r minitest/autorun hello_world.rb
    Run options: --seed 21214

    # Running:

    .F

    Finished in 0.001177s, 1699.2353 runs/s, 1699.2353 assertions/s.

      1) Failure:
    TestHelloWorld#test_hello_to_matz [hello_world.rb:20]:
    Failed assertion, no message given.

    2 runs, 2 assertions, 1 failures, 0 errors, 0 skips
{terminal}

Not only does our original test pass again, but our new test `test_hello_to_matz` no longer errors. We are making progress. But our new test isn't passing yet, so we have some work to do. Let's update the HelloWorld#hello method to make it pass.

Make it Green
-------------

When we were writing our tests we put ourselves in the mindset of the consumers of our code. We thought through what API we wanted to use, not just what was easy to expose. Now that we are changing our code I want to change mindset once again. I want to put on the personality of the Worst Programmer in the World. I want to take any and all shortcuts needed to get the test to pass. I want to intentionally write bad or buggy code.

How would the Worst Programmer in the World make this test pass? Probably hard code the implementation. So let's do that.

    class HelloWorld
      def hello(name = nil)
        if name.nil?
          "Hello World!"
        else
          "Hello Matz!"
        end
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

      def test_hello_to_matz
        assert("Hello Matz!" == @hello_world.hello("Matz"))
      end

      def teardown
        @hello_world = nil
      end
    end
{hello_world.rb}

What an awful solution! Surely Minitest wouldn't allow something as bad as that to pass, right?

    $ ruby -r minitest/autorun hello_world.rb
    Run options: --seed 19258

    # Running:

    ..

    Finished in 0.001101s, 1816.5304 runs/s, 1816.5304 assertions/s.

    2 runs, 2 assertions, 0 failures, 0 errors, 0 skips
{terminal}

It passes! Our terrible code passes! But the code feels dirty, doesn't it? Time for another mindset change. We need some Quality Assurance, so let's put ourselves in the mindset of someone trying to expose edge cases in our code. Let's create a few more tests sending "Ryan", "Aaron", and "Eric" into our greeting:

    class HelloWorld
      def hello(name = nil)
        if name.nil?
          "Hello World!"
        else
          "Hello Matz!"
        end
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

      def test_hello_to_matz
        assert("Hello Matz!" == @hello_world.hello("Matz"))
      end

      def test_hello_to_ryan
        assert("Hello Ryan!" == @hello_world.hello("Ryan"))
      end

      def test_hello_to_aaron
        assert("Hello Aaron!" == @hello_world.hello("Aaron"))
      end

      def test_hello_to_eric
        assert("Hello Eric!" == @hello_world.hello("Eric"))
      end

      def teardown
        @hello_world = nil
      end
    end
{hello_world.rb}

Now that we have three new tests, we should have three new failures:

    $ ruby -r minitest/autorun hello_world.rb
    Run options: --seed 33140

    # Running:

    F.F.F

    Finished in 0.001300s, 3846.1538 runs/s, 3846.1538 assertions/s.

      1) Failure:
    TestHelloWorld#test_hello_to_eric [hello_world.rb:36]:
    Failed assertion, no message given.


      2) Failure:
    TestHelloWorld#test_hello_to_ryan [hello_world.rb:28]:
    Failed assertion, no message given.


      3) Failure:
    TestHelloWorld#test_hello_to_aaron [hello_world.rb:32]:
    Failed assertion, no message given.

    5 runs, 5 assertions, 3 failures, 0 errors, 0 skips
{terminal}

Okay, what does the Worst Programmer in the World do now? Probably write some more terrible code.

    class HelloWorld
      def hello(name = nil)
        if name == "Matz"
          "Hello Matz!"
        elsif name == "Ryan"
          "Hello Ryan!"
        elsif name == "Aaron"
          "Hello Aaron!"
        elsif name == "Eric"
          "Hello Eric!"
        else
          "Hello World!"
        end
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

      def test_hello_to_matz
        assert("Hello Matz!" == @hello_world.hello("Matz"))
      end

      def test_hello_to_ryan
        assert("Hello Ryan!" == @hello_world.hello("Ryan"))
      end

      def test_hello_to_aaron
        assert("Hello Aaron!" == @hello_world.hello("Aaron"))
      end

      def test_hello_to_eric
        assert("Hello Eric!" == @hello_world.hello("Eric"))
      end

      def teardown
        @hello_world = nil
      end
    end
{hello_world.rb}

Awful. Does it pass?

    $ ruby -r minitest/autorun hello_world.rb
    Run options: --seed 47356

    # Running:

    .....

    Finished in 0.001368s, 3654.9708 runs/s, 3654.9708 assertions/s.

    5 runs, 5 assertions, 0 failures, 0 errors, 0 skips
{terminal}

It should feel like you are trolling yourself. The QA mindset finds the edges of what is acceptable behavior while the Worst Programmer mindset takes every short cut and intentionally avoids any sense of design. Why? The purpose of writing these tests is to design the API to be consumed with an acceptable amount of test coverage so we know if it works. Now we are ready to move on to the final phase of the TDD Cycle.

Make it Pretty
--------------

Now that the Customer and QA and Worst Programmer mindsets have gone rounds we have a suite of tests that demonstrate the API and show how it is intended to work. Now we can focus on changing the code so that it is correct. This step is also known as the Refactoring step, because we want to be able to make changes to how our code is implemented without changing the API we have defined. Let's switch to the Engineer mindset and start thinking about what improvements we should make to the code.

Right now HelloWorld#hello has five conditional branches. My Engineer mindset doesn't like that. How can we change the implementation without changing the API? We could use a different default value and use string interpolation.

    class HelloWorld
      def hello(name = "World")
        "Hello #{name}!"
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

      def test_hello_to_matz
        assert("Hello Matz!" == @hello_world.hello("Matz"))
      end

      def test_hello_to_ryan
        assert("Hello Ryan!" == @hello_world.hello("Ryan"))
      end

      def test_hello_to_aaron
        assert("Hello Aaron!" == @hello_world.hello("Aaron"))
      end

      def test_hello_to_eric
        assert("Hello Eric!" == @hello_world.hello("Eric"))
      end

      def teardown
        @hello_world = nil
      end
    end
{hello_world.rb}

Much better! Does it continue to pass?

    $ ruby -r minitest/autorun hello_world.rb
    Run options: --seed 14436

    # Running:

    .....

    Finished in 0.001117s, 4476.2757 runs/s, 4476.2757 assertions/s.

    5 runs, 5 assertions, 0 failures, 0 errors, 0 skips
{terminal}

Yep, it passes and all is right with the world. If I had ideas of design patterns to use I would implement them now. But the key in this phase is to only change the code, not the tests. If you change the tests in this phase you are redesigning and not refactoring.

Make (the tests) Pretty
-----------------------

I said that the Refactoring step was to make changes to the code, and that's true. But once I have all my code changes made I also like to revisit the design of the tests. We have one test for the default `hello` greeting, and four for the custom greeting. Perhaps we can rewrite the custom tests to not be as verbose by iterating over an array of names?

    class HelloWorld
      def hello(name = "World")
        "Hello #{name}!"
      end
    end

    gem     "minitest"
    require "minitest"

    class TestHelloWorld < Minitest::Test
      def setup
        @hello_world = HelloWorld.new
      end

      def test_hello_default
        assert("Hello World!" == @hello_world.hello)
      end

      def test_hello_custom
        names = %w{Matz Ryan Aaron Eric}
        names.each do |name|
          assert("Hello #{name}!" == @hello_world.hello(name))
        end
      end

      def teardown
        @hello_world = nil
      end
    end

Much better! While there is a bit more code to parse in the `test_hello_custom` test, it verifies the same values while not being as verbose. And the tests still pass cleanly:

    $ ruby -r minitest/autorun hello_world.rb
    Run options: --seed 50853

    # Running:

    ..

    Finished in 0.001144s, 1748.2517 runs/s, 4370.6294 assertions/s.

    2 runs, 5 assertions, 0 failures, 0 errors, 0 skips

We can make even more improvements to the tests. One thing that bothered my QA mindset was that we were dealing with a fixed list of names to test. I want to keep the Worst Programmer mindset honest, and the only way I can think of doing that is by giving it random names that the Worst Programmer can't anticipate. So let's create a new *support method* to generate a random name and add a couple random names to the list.

    class HelloWorld
      def hello(name = "World")
        "Hello #{name}!"
      end
    end

    gem     "minitest"
    require "minitest"
    require "securerandom"

    class TestHelloWorld < Minitest::Test
      def setup
        @hello_world = HelloWorld.new
      end

      def test_hello_default
        assert("Hello World!" == @hello_world.hello)
      end

      def test_hello_custom
        names = %W{Matz Ryan Aaron Eric #{random_name} #{random_name}}
        names.each do |name|
          assert("Hello #{name}!" == @hello_world.hello(name))
        end
      end

      def random_name
        SecureRandom.hex(8)
      end

      def teardown
        @hello_world = nil
      end
    end

And the tests still pass.

    $ ruby -r minitest/autorun hello_world.rb
    Run options: --seed 28991

    # Running:

    ..

    Finished in 0.001130s, 1769.9115 runs/s, 6194.6903 assertions/s.

    2 runs, 7 assertions, 0 failures, 0 errors, 0 skips

Again, I only revisit the test design after all the tests pass cleanly and all code under test is refactored and my Engineer mindset is satisfied.

Conclusion
----------

You've now written tests to go from **Error** to **Failure** to **Passing** to **Refactoring**. This is the TDD Cycle. Use the feedback the tests provide to make changes. The tests provide coverage for the API you are exposing to your software's consumers. This will give you the confidence to make improvements to your software by providing reassurance that you aren't breaking existing users.

The TDD Cycle also promotes looking at your code from difference mindsets. You look at your code from multiple perspectives, from the consumer to the maintainer, allowing you to entertain both your creativity and your focus/completeness/something. The more you practice TDD, the better at these mindsets you get, and the better code you will write.
