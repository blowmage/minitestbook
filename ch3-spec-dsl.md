Chapter 3 - Minitest::Spec DSL
==============================

In chapter 1 we looked at the Minitest API and using `Minitest::Test` to create *Test Classes* and *Test Methods* using *Assertion Methods* and *Support Methods*. This style of testing may seem primitive to some who prefer using a high level spec DSL instead of an API. Minitest also provides a spec DSL, but to understand the spec DSL you must first understand the Minitest API. So I encourage you to read Chapter 1 if you skipped it in your excitement.

Using a spec DSL is also very popular with practitioners of Behavior Driven Development, or BDD, a variation of TDD. In BDD the spec DSL is used to create tests that read as a software specification. Minitest's spec DSL is built on top of Minitest's API and provides this abstraction with minimal DIFFERENCE(?) [(Jason) changes? confusion?].

To demonstrate the spec DSL, let's start with the `test_hello_world.rb` file created from Chapters 1 and 2. We will convert this file from using the Minitest API to the spec DSL. Here are the contents of the file we are starting with:

    [
    (Jason) Should probably include the class under test, for completeness.
    I personally don't like using require_relative, and could be confusing here.
    Might be able to have the full file here, code and tests, and for the rest of
    the code blocks in this chapter, simply include the test code.
    ]
    
    gem     "minitest"
    require "minitest"
    require "securerandom"
    require_relative "hello_world.rb"

    class TestHelloWorld < Minitest::Test
      def setup
        @hello_world = HelloWorld.new
      end

      def test_hello_default
        assert_equal "Hello World!", @hello_world.hello
      end

      def test_hello_custom
        names = %W{Matz Ryan Aaron Eric #{random_name} #{random_name}}
        names.each do |name|
          assert_equal "Hello #{name}!", @hello_world.hello(name)
        end
      end

      def random_name
        SecureRandom.hex(8)
      end

      def teardown
        @hello_world = nil
      end
    end
{test\_hello\_world.rb}

The `describe` Block
--------------------

The spec DSL is comprised of a series of nested blocks. Instead of creating a *Test Class* explicitly, we can use a **`describe` block** to do this for us. We can replace the line `class TestHelloWorld < Minitest::Test` with a describe:

    gem     "minitest"
    require "minitest"
    require "securerandom"
    require_relative "hello_world.rb"

    describe HelloWorld do
      def setup
        @hello_world = HelloWorld.new
      end

      def test_hello_default
        assert_equal "Hello World!", @hello_world.hello
      end

      def test_hello_custom
        names = %W{Matz Ryan Aaron Eric #{random_name} #{random_name}}
        names.each do |name|
          assert_equal "Hello #{name}!", @hello_world.hello(name)
        end
      end

      def random_name
        SecureRandom.hex(8)
      end

      def teardown
        @hello_world = nil
      end
    end
{test\_hello\_world.rb}

The `describe` block can be passed the constant of the class you are testing, or a string, or any object that can be converted to a string, like a symbol. That said, a class constant it preferred where possible.

We can use nested `describe` blocks to narrow the focus of our tests. This focus can be about a specific method under test or it can be a scenario such as collaborator being valid or invalid, or permissions being present or not present. In our case, both of our existing *Test Methods* are specific to the `hello` method, so let's create a nested `describe` block that is focused on that method:

    gem     "minitest"
    require "minitest"
    require "securerandom"
    require_relative "hello_world.rb"

    describe HelloWorld do
      def setup
        @hello_world = HelloWorld.new
      end

      describe :hello do
        def test_hello_default
          assert_equal "Hello World!", @hello_world.hello
        end

        def test_hello_custom
          names = %W{Matz Ryan Aaron Eric #{random_name} #{random_name}}
          names.each do |name|
            assert_equal "Hello #{name}!", @hello_world.hello(name)
          end
        end
      end

      def random_name
        SecureRandom.hex(8)
      end

      def teardown
        @hello_world = nil
      end
    end
{test\_hello\_world.rb}

Under the covers Minitest is creating a *Test Class* with the contents of each `describe` block. The *Test Class* inherits from the `describe` block it is nested under, or the default `Minitest::Spec` class if it is not nested. So the *Test Class* created by the `describe :hello` block inherits from the *Test Class* created by the `describe HelloWorld` block, which inherits from `Minitest::Spec` (which inherits from `Minitest::Test`).

The `describe` block is just another way to create a *Test Class* in your tests. The output of the spec DSL are objects using the Minitest API.

The `it` Block
--------------

You can create *Test Methods* using the spec DSL, just as you can create *Test Classes*. *Test Methods* are created using the **`it` block**, which must be nested inside of a `describe` block, the same way a *Test Method* must be located inside a *Test Class*. The `it` block takes a string that can provide more descriptive description (?!?) than the name of the *Test Method*:

    gem     "minitest"
    require "minitest"
    require "securerandom"
    require_relative "hello_world.rb"

    describe TestHelloWorld do
      def setup
        @hello_world = HelloWorld.new
      end

      describe :hello do
        it "gives a default greeting" do
          assert_equal "Hello World!", @hello_world.hello
        end

        it "gives a custom greeting" do
          names = %W{Matz Ryan Aaron Eric #{random_name} #{random_name}}
          names.each do |name|
            assert_equal "Hello #{name}!", @hello_world.hello(name)
          end
        end
      end

      def random_name
        SecureRandom.hex(8)
      end

      def teardown
        @hello_world = nil
      end
    end
{test\_hello\_world.rb}

Under the covers Minitest is creating a *Test Method* that uses the string provided to the `it` block. So the contents of the `it "gives a default greeting"` block is used to create a method named "`test_0001_gives a default greeting`" and contents of the `it "gives a custom greeting"` block is used to create a method named "`test_0002_gives a custom greeting`".

The `it` block is just another way to create a *Test Methods* in your tests. The output of the spec DSL are methods using the Minitest API.

Expectations
------------

Instead of passing calculated values to the main `assert` method, or using special case assertions such as `assert_equal` or `assert_match`, we can use call the assertions using an alternate API(?) called **expectations**.

[(Jason) No test methods outside of `assert` have been mentioned yet. Should probably add a section that lists these out before talking about how they switch over in Spec form.]

The common order of arguments for assertions is expected, then actual. Consider the argument names for Minitest's `assert_equal` method:

    def assert_equal exp, act, msg = nil
      msg = message(msg, "") { diff exp, act }
      assert exp == act, msg
    end

In the assertion method the position of the arguments is important, but expectations are called on the actual value eliminating the need to remember the positioning. In our tests, the `assert_equal` assertion:

    assert_equal "Hello World!", @hello_world.hello

Can be written with the `must_equal` expectation:

    @hello_world.hello.must_equal "Hello World!"

Our revised test is now:

    gem     "minitest"
    require "minitest"
    require "securerandom"
    require_relative "hello_world.rb"

    describe TestHelloWorld do
      def setup
        @hello_world = HelloWorld.new
      end

      describe :hello do
        it "gives a default greeting" do
          @hello_world.hello.must_equal "Hello World!"
        end

        it "gives a custom greeting" do
          names = %W{Matz Ryan Aaron Eric #{random_name} #{random_name}}
          names.each do |name|
            @hello_world.hello(name).must_equal "Hello #{name}!"
          end
        end
      end

      def random_name
        SecureRandom.hex(8)
      end

      def teardown
        @hello_world = nil
      end
    end
{test\_hello\_world.rb}

Under the covers Minitest is calling *Assertion Methods* from the *Expectations*. *Expectations* are just another way to call *Assertion Methods* in your tests. The output of the spec DSL are methods using the Minitest API.

The `before` and `after` Blocks
---------------------------

The spec DSL has similar analogs for the `setup` and `teardown` support methods. They are expressed as `before` and `after` blocks. The contents of the `setup` and `teardown` support methods will transfer directly to the `before` and `after` blocks.

    gem     "minitest"
    require "minitest"
    require "securerandom"
    require_relative "hello_world.rb"

    describe TestHelloWorld do
      before do
        @hello_world = HelloWorld.new
      end

      describe :hello do
        it "gives a default greeting" do
          @hello_world.hello.must_equal "Hello World!"
        end

        it "gives a custom greeting" do
          names = %W{Matz Ryan Aaron Eric #{random_name} #{random_name}}
          names.each do |name|
            @hello_world.hello(name).must_equal "Hello #{name}!"
          end
        end
      end

      def random_name
        SecureRandom.hex(8)
      end

      after do
        @hello_world = nil
      end
    end
{test\_hello\_world.rb}

The `let` Block
---------------

Since the test was only using the `setup` and `teardown` methods to initialize an instance variable in our *Test Class*, the spec DSL provides a way to declare this outside of the the `before` and `after` blocks. This is the `let` block.

The `let` block will create a support method that returns the result of the given block. It will execute the block only once per test. While it can be called several times in a test the block will only be executed the first time. Unlike the `setup` method or the `before` block it is lazy evaluated so if it is never called the block will never be invoked.

    gem     "minitest"
    require "minitest"
    require "securerandom"
    require_relative "hello_world.rb"

    describe TestHelloWorld do
      let(:hello_world) { HelloWorld.new }

      describe :hello do
        it "gives a default greeting" do
          hello_world.hello.must_equal "Hello World!"
        end

        it "gives a custom greeting" do
          names = %W{Matz Ryan Aaron Eric #{random_name} #{random_name}}
          names.each do |name|
            hello_world.hello(name).must_equal "Hello #{name}!"
          end
        end
      end

      def random_name
        SecureRandom.hex(8)
      end
    end
{test\_hello\_world.rb}

Under the covers of the `let` block Minitest is creating several *Support Methods* for you. Minitest creates code similar to the following:

    class TestHelloWorld < Minitest::Test
      def setup
        @hello_world = nil
      end

      def hello_world
        @hello_world ||= HelloWorld.new
      end

      def teardown
        @hello_world = nil
      end
    end
{test\_hello\_world.rb}

This is one of the advantages [(Jason) I assume more will go here?]

The `before` and `after` and `let` blocks are more ways to create a *Support Methods* in your tests. The output of the spec DSL are methods using the Minitest API.

Conclusion
----------

You've now converted your test from the Minitest API to the spec DSL. 

You've created *Test Classes* using **`describe` blocks**, created *Test Methods* using **`it` blocks**, called *Assertion Methods* using **expectations**, and created *Support Methods* using **`before`** and **`after`** and **`let` blocks**.

The spec DSL allows for an english-like/high-level abstraction for creating objects and methods using the Minitest API. Minitest tests can use a combination of the spec DSL and the Minitest API. You can use `it` blocks with *Test Classes*, and *Assertions* with `let` blocks. The spec DSL generates objects and methods according to the Minitest API that Minitest uses.
