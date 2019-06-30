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

```Ruby
class Robot
  attr_reader :name

  def initialize(name)
    @name = name
  end

  def introduce
    Greeting.say_hello(name)
  end
end
```

If our code has many dependencies it can make our code difficult to read and difficult to update. A small change in a class like `Greeting` may have flow on effects to other distant classes meaning that changing one part of your code will break other functionality. One technique for reducing the negative impact of dependencies in your code is - you guessed it - *dependency injection*.

We can use dependency injection to make our code more flexible, extensible and reduce the likelihood that changes to the way the `Greeting` class is implemented will cause problems in our  `Robot` class.

## What's dependency injection?

##### Dependency injection lets us pass an object's dependencies to it as an argument.

In the example below, we pass a new `Greeting` object to the `Robot` class in the initializer as a default argument. Our `introduce` method now no longer depends on `Greeting` directly but rather relies on being passed an argument we have called `greeting` which can respond to the `say_hello` argument.

In the example below, the class `Robot` knows the following:
- It will be passed an object when it is created
- This object will respond to the method `say_hello`

The `Greeting` class is no longer a dependency, instead it is being injected into the `Robot` class.

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

When we create our first robot and only pass in the name Bleep the default `Greeting` class will be used. However, when we create our new robot Bloop and pass in a new `GreetingInPolish` object Bloop now responds to the `introduce` method with a Polish greeting.

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

Using dependency injection in our testing helps us isolate the code under test and to highlight when a class may have too many dependencies.

We can isolate our unit tests using dependency injection by creating a dummy class that exists only in the test environment and injecting it as an argument into our class - much as we did with our `GreetingInPolish` class.

Additionally, dependency injection can help us to test functionality that we don't actually want to run as part of the testing process.

For example, imagine that Bleep and Bloop can fly, we might want to test that the `Robot` class is able to call a `blast_off` method without actually making our robots take to the skies every time we run our test suite.

# NOTE: You can also tell if you have a few too many dependencies because you'll have way too much stuff to pass in!

## How can we use dependency injection in testing?

In the example below we have removed the `introduce` method and implemented a `fly` method.

When it comes to testing the `fly` method we don't want to use the real `FlightController` class or `blast_off` method. Rather, we want to isolate our testing of `fly`. For this unit test all we want to know is whether fly sends the correct messages. We don't actually need to know if the receiving class `FlightController` responds correctly, because we will test that elsewhere, for example, in a unit test on the `blast_off` method or in an integration test.

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
For an introduction to Dependency Injection in Java outside of the context of testing.
https://www.freecodecamp.org/news/a-quick-intro-to-dependency-injection-what-it-is-and-when-to-use-it-7578c84fa88f/

For an introduction to dependency injection in Ruby using the dry-rb libraries.
https://medium.com/@Bakku1505/introduction-to-dependency-injection-in-ruby-dc238655a278

Explanation of Dependency Injection focused on examples
http://rubyblog.pro/2016/10/ruby-dependency-injection

Explanation of dependency injection in the context of Rails focused on examples.
https://www.codewithjason.com/dependency-injection-can-make-rails-tests-easier/

A general overview of unit testing in Ruby
https://blog.codeship.com/unit-testing-in-ruby/
