# Test doubles in RSpec

##### Double is an umbrella term for an object that stands in for a real object in a test.

A common analogy used for describing test doubles is that they are like stunt doubles working on a film shoot. Like a stunt double, a test double can stand in for the real code under test when we want particular kinds of behaviours that the real code cannot easily provide. Most importantly, using test doubles helps us isolate unit tests by standing in for classes and methods that are **undefined, unavailable, expensive or difficult** to work with.

The language around these ideas is often used inexactly and interchangeably. In particular, the word "mock" is often used as a generic term to refer to test doubles more broadly. For the purposes of this explanation we're going to use some definitions of types of test doubles from an article by [Martin Fowler](http://martinfowler.com/articles/mocksArentStubs.html).

This is not an exhaustive list of types of test doubles but these are the ones we will discuss in future posts.

**Dummy** – an object that is only being used as a placeholder, it doesn't and can't respond to any messages. You use a dummy when you need to pass an object into a method as an argument but you don't plan on using any of its functionality in your test.

**Stub** – an object can have a method on it stubbed, this stubbed method will be setup to provide a canned response when called. You can stub out methods on a test double or on a real object.

**Mock** – an object that has been given a method or list of methods that it must/must not receive during the test to allow the test to pass.

**Spy** – an object that has been set up to record all the methods it receives during a test, it can then report back on what it received during the expectation part of the test.

## Resources
RSpec Documentation:
https://relishapp.com/rspec/rspec-mocks/docs/basics/test-doubles

Doubles in RSpec:
https://www.tutorialspoint.com/rspec/rspec_test_doubles.htm

An introduction to stubs, mocks and spies with clear examples:
https://about.futurelearn.com/blog/stubs-mocks-spies-rspec

Chapter 13, Effective Testing with RSpec 3:
https://learning.oreilly.com/library/view/effective-testing-with/9781680502770/f_0103.xhtml#chp.understanding-test-doubles
