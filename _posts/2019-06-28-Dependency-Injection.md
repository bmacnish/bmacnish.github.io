---
layout: post
title: Dependency Injection
categories: [coding, code, learn to code, ruby, rspec, testing, unit test, dependency, dependencies, dependency injection]

---
#### Dependency injection can be used when writing and when testing code. Here, we're going to briefly discuss what dependency injection is and then go into detail on how we can use it to our advantage when writing unit tests for our code.

## What's a dependency?

##### A dependency exists when one class knows about and uses methods from another class. When this happens we say the first class has a *dependency* on the second class.

In the example below, the class `Robot` knows the following about the class `Greeting`:
- A class called `Greeting` exists
- `Greeting` implements the method `say_hello`
- `say_hello` takes one argument

Therefore, `Robot`, *depends* on `Greeting`.

{% highlight ruby %}
class Robot
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def introduce
    Greeting.say_hello(name)
  end
end
{% endhighlight %}

If our code has many dependencies it can make our code difficult to read and update because a small change in a class like `Greeting` may have flow on effects to other distant classes. When we have too many dependencies changing one part of your code will break other unrelated functionality. One technique for reducing the negative impact of dependencies in your code is - you guessed it - *dependency injection*.

We can use dependency injection to make our code more flexible, extensible and reduce the likelihood that changes to the way the `Greeting` class is implemented will cause problems in our `Robot` class.

## What's dependency injection?

##### Dependency injection lets us pass an object's dependencies to it as an argument.

In the example below, we pass a new `Greeting` object to the `Robot` class as a default argument. Our `introduce` method now no longer depends on `Greeting` directly but rather relies on being passed an argument we have called `greeting` which can respond to the `say_hello` argument.

In the example below, the class `Robot` now knows the following:
- It will be passed an object when it is created
- This object will respond to the method `say_hello`

The `Greeting` class is now being *injected* into the `Robot` class.

{% highlight ruby %}
class Robot

  attr_reader :name, :greeting

  def initialize(name, greeting = Greeting.new)
    @name = name
    @greeting = greeting
  end

  def introduce
    greeting.say_hello(name)
  end
end


class Greeting
  def say_hello(name)
    "Hello, my name is #{name}"
  end
end


puts Robot.new("Bleep").introduce
# Hello, my name is Bleep
{% endhighlight %}

But there's more! Injecting our `Greeting` class lets us also inject any other class that responds to the 'say_hello' method. To demonstrate this, we've created a new class, `GreetingInPolish`.

When we create our first robot and only pass in the name Bleep the default `Greeting` class will be used. However, when we create our second robot Bloop and pass in a new `GreetingInPolish` object Bloop will respond to the `introduce` method with a Polish greeting.

This is possible because both `Greeting` and `GreetingInPolish` respond to the method `say_hello`.

{% highlight ruby %}
class Robot

  attr_reader :name, :greeting

  def initialize(name, greeting = Greeting.new)
    @name = name
    @greeting = greeting
  end

  def introduce
    greeting.say_hello(name)
  end
end


class Greeting

  def say_hello(name)
    "Hello, my name is #{name}"
  end
end


class GreetingInPolish

  def say_hello(name)
    "Cześć, mam na imię #{name}"
  end
end


puts Robot.new("Bleep").introduce
# Hello, my name is Bleep

puts Robot.new("Bloop", GreetingInPolish.new).introduce
# Cześć, mam na imię Bloop
{% endhighlight %}

## Why use dependency injection in testing?

Using dependency injection in our testing helps us to:
- isolate the code under test
- test functionality that we don't want to actually run
- highlight when a class may have too many dependencies

#### Isolation
We can isolate our unit tests using dependency injection by creating a dummy class that exists only in the test environment and injecting it as an argument into our class - much as we did with our `GreetingInPolish` class.

#### Testing code we don't want to run
Additionally, dependency injection can help us to test functionality that we don't actually want to run as part of the testing process.

For example, imagine that Bleep and Bloop can fly, we might want to test that the `Robot` class is able to call a `blast_off` method without actually making our robots take to the skies every time we run our test suite.

#### Highlighting when we have too many dependencies
Because using dependency injection requires us to pass dependencies as an argument to our initialize method it becomes quickly apparent when we have a large number of dependencies. This can provide a hint that the structure of our code needs rethinking and that a class may have too many dependencies.

## How can we use dependency injection in testing?

In the example below, we have removed the `introduce` method from our `Robot` class and implemented a new `fly` method.

When it comes to testing the `fly` method we don't want to use the *real* `FlightController` class or `blast_off` method. We want to be able to test our robot in a lab setting without putting a hole in the roof!

 For this unit test all we want to know is whether `fly` sends the correct messages. We don't actually need to know if the receiving class `FlightController` responds correctly, because we will test that elsewhere, for example, in a unit test on the `blast_off` method or in an integration test.

So, inside our RSpec test we create a new class that only exists within our testing environment `TestFlightController` that implements a test version of `blast_off`. When we call `fly` on our testing robot Bloop we can then expect to receive the testing message: `"This is a test message: 3 ... 2 ... 1 ... BLAST OFF!"`

When our test passes, we now know that `fly` sends the correct message to the `flight_controller` without actually blasting our robot into the stratosphere!

{% highlight ruby %}
class Robot

  attr_reader :name, :flight_controller

  def initialize(name, flight_controller = FlightController.new)
    @name = name
    @flight_controller = flight_controller
  end

  def fly
    flight_controller.blast_off
  end
end

class FlightController
  def blast_off
    "3 ... 2 ... 1 ... BLAST OFF!"
  end
end

puts Robot.new("Bleep").fly
# 3 ... 2 ... 1 ... BLAST OFF!

RSpec.describe 'Robot' do

  class TestFlightController
    def blast_off
      "This is a test message: 3 ... 2 ... 1 ... BLAST OFF!"
    end
  end

  describe "#fly" do
    it 'prints out a blast off message' do
      expect(Robot.new("Bloop", TestFlightController.new).fly).to eql("This is a test message: 3 ... 2 ... 1 ... BLAST OFF!")
    end
  end
end
{% endhighlight %}

## Resources:
For an introduction to Dependency Injection in Java outside of the context of testing: https://www.freecodecamp.org/news/a-quick-intro-to-dependency-injection-what-it-is-and-when-to-use-it-7578c84fa88f/

Explanation of Dependency Injection focused on example: http://rubyblog.pro/2016/10/ruby-dependency-injection

A general overview of unit testing in Ruby: https://blog.codeship.com/unit-testing-in-ruby/
