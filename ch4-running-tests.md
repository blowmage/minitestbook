Running Tests
=============

So far we have been passing `-r minitest/autorun` to ruby when we want to run the tests in a file. But what if we want to run all the tests in our application? Or what if we want to run just one test method in a file?

Minitest provides an object called a Runner that will run the tests for us and print the output. This makes running tests very easy. All we need to do is to load the runner with our code. Let's do this by passing `-r minitest/autorun` to Ruby to tell it to load the runner as we load our code: