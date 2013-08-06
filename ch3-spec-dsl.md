Chapter 3 - Minitest::Spec DSL
==============================

In chapter 1 we looked at the Minitest API and using `Minitest::Test` to create *test classes* and *test methods* using *assertion methods* and *support methods*. This style of testing may seem primitive to some who prefer using a high level spec DSL instead of an API. Minitest also provides a spec DSL, but to understand the spec DSL you must first understand the Minitest API. So I encourage you to read chapter 1 if you skipped it in your excitement.

Using a spec DSL is also very popular with practitioners of BDD, a variation of TDD. In BDD the spec DSL is used to create tests that can read as a software specification. Minitest's spec DSL is built on top of Minitest's API and provides this abstraction with minimal DIFFERENCE(?).

To demonstrate the spec DSL, let's start with the `test_hello_world.rb` file created from chapters 1 and 2. We will convert this file from using the Minitest API to the spec DSL. Here are the contents of the file we are starting with:

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

The spec DSL is comprised of a series of nested blocks. Instead of creating a *test class* explicitly, we can use a **`describe` block** to do this for us. We can replace the line `class TestHelloWorld < Minitest::Test` with a describe:

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

The `describe` block can be passed the constant of the class you are testing, or a string, or any object that can be converted to a string, like a symbol. But for the most part a class constant it preferred when possible.

We can use nested `describe` blocks to narrow the focus of our tests. This focus can be about a specific method under test, or it can be a scenario such as collaborator being valid or invalid, or permissions being present or not present. In our case, both of our existing *test methods* are specific to the `hello` method, so let's create a nested `describe` block that is focused on that method:

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

Under the covers Minitest is creating a *test class* with the contents of each `describe` block. The *test class* inherits from the `describe` block it is nested under, or the default `Minitest::Spec` class if it is not nested. So the *test class* created by the `describe :hello` block inherits from the *test class* created by the `describe HelloWorld` block, which inherits from `Minitest::Spec` (which inherits from `Minitest::Test`).

The `describe` block is just another way to create a *test class* in your tests. The output of the spec DSL are objects using the Minitest API.

The `it` Block
--------------

You can create *test methods* using the spec DSL, just as you can create *test classes*. *Test methods* are created using the **`it` block**, which must be nested inside of a `describe` block, the same way a *test method* must be located inside a *test class*. The `it` block takes a string that can provide more descriptive description (?!?) than the name of the *test method*:

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

Under the covers Minitest is creating a *test method* that uses the string provided to the `it` block. So the contents of the `it "gives a default greeting"` block is used to create a method named "`test_0001_gives a default greeting`" and contents of the `it "gives a custom greeting"` block is used to create a method named "`test_0002_gives a custom greeting`".

The `it` block is just another way to create a *test methods* in your tests. The output of the spec DSL are methods using the Minitest API.

Expectations
------------

Instead of passing calculated values to the main `assert` method, or using special case assertions such as `assert_equal` or `assert_match`, we can use call the assertions using an alternate API(?) called **expectations**.

The common order of arguments for assertions is expected, then actual. Consider the argument names for the `assert_equal` method as defined in Minitest:

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

Under the covers Minitest is calling *assertion methods* from the *expectations*. *Expectations* are just another way to call *assertion methods* in your tests. The output of the spec DSL are methods using the Minitest API.

The `before` and `after` Blocks
---------------------------

The spec DSL has analogs for the `setup` and `teardown` support methods. They are expressed as `before` and `after` blocks. The contents of the `setup` and `teardown` support methods will transfer directly to the `before` and `after` blocks.

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

Since the test was only using the `setup` and `teardown` methods to initialize an instance variable in our *test class*, the spec DSL provides a way to declare this outside of the the `before` and `after` blocks. This is the `let` block.

The `let` block will create a support method that returns the result of the given block. And it will execute the block only once per test. So while it can be called several times in a test the block will only be executed the first time. And unlike the `setup` method or the `before` block it is lazy evaluated so if it is never called the block will never be invoked.

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

Under the covers of the `let` block Minitest is creating using several support methods. Minitest creates code similar to the following:

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

This is one of the advantages 

The `before` and `after` and `let` blocks are more ways to create a *support methods* in your tests. The output of the spec DSL are methods using the Minitest API.

Conclusion
----------

You've now converted your test from the Minitest API to the spec DSL. 

You've created *test classes* using **`describe` blocks**, created *test methods* using **`it` blocks**, called *assertion methods* using **expectations**, and created *support methods* using **`before`** and **`after`** and **`let` blocks**.

The spec DSL allows for an english-like/high-level abstraction for creating objects and methods using the Minitest API. Minitest tests can use a combination of the spec DSL and the Minitest API. You can use `it` blocks with *test classes*, and *assertions* with `let` blocks. The spec DSL generates objects and methods according to the Minitest API that Minitest uses.
