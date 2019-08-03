---
layout: post
title: Test Mocks
categories: [coding, code, learn to code, ruby, rspec, testing, unit test, double, dummy, stubs, mocks]
---
#### A mock is an object that has been given a method or list of methods that it must or must not receive during a test to allow the test to pass.

## Syntax
{% highlight ruby %}
expect(object).to receive(:method)
#OR
expect(object).to receive(:method).with(specific_argument)
#OR
expect(object).to receive(:method).with(specific_argument).and_return(return_value)
{% endhighlight %}

## Why would you use a mock?
As we use stubs to test return values we use mocks to test if messages have been sent or received. Generally, we use mocks to test methods that do something rather than methods that return something, methods that **command** rather than **query**.

Mocks are used to ask the following:

>“Did we tell others the right thing to do, with the right information, and exactly the right amount of times that we need?” -
 [Ruby Guides](https://www.rubyguides.com/2018/10/rspec-mocks/)

## Code example

In the following example, we are going to give our robot a self-destruct mechanism because we've read plenty of dystopian sci-fi and know it won't be long until Fleep turns on us.

{% highlight ruby %}
class Robot

  attr_reader :name

  def initialize(name)
    @name = name
  end

  def self_destruct
    Terminate.initiate
  end
end

class Terminate
  def self.initiate
    "Self destruct sequence initiated."
  end
end
{% endhighlight %}
We have added a `self_destruct` method to our main `Robot` class which calls the class method `initiate` on our new class `Terminate`.

Our problem is, we want to test our `initiate` method to make sure it is working properly but we don't want our robot to actually self destruct. This is the same issue we encountered in our stubbing example when we wanted to test `flight controller`. Let's briefly discuss why these two examples are different.

In our stubbing example, we were testing an api call to a weather service. We expected to get information back and were therefore making a **query**. In our current mocking example, we expect a (rather drastic) action to be taken in response to our method call and are therefore executing a **command**.

The differences can be a little subtle but in essence a query method expects to **get information** when it runs while a command method expects to **change the state of something** when it runs.

So, we have a piece of code and we know three things about it:
 - we want to test it
 - we don't want to run it
 - it's a command method

A perfect situation to try out mocking!

## Test example
In our test below, we use the RSpec mocking syntax to set up the expectation that `Terminate` will receive the `initiate` method. RSpec is now watching `Terminate` and will not allow the test to pass **unless** it receives `initiate`.

We will then create a new robot Fleep, and call `self_destruct`. If the test passes we know that `initiate` was received by `Terminate`.

To reassure ourselves that our poor robot has not really been blown up we can double check that RSpec is definitely only checking that `initiate` has been received and is not running the real `initiate` code.

To do this we set up the test to provide the following return value, `'The test self destruct sequence has run'`.

Rather than the return value for the real sequence, `'Self destruct sequence initiated'`.

{% highlight ruby %}
RSpec.describe 'Terminate' do
  it 'initiates self destruct' do
  fleep = Robot.new("Fleep")

  expect(Terminate).to receive(:initiate).and_return("The test self destruct sequence has run")

  fleep.self_destruct
  end
end
{% endhighlight %}

## That's a bit weird ...

You might have noticed that our test is structured differently to other tests.

You are probably used to seeing tests structured in three steps.

1. The setup
2. The execution
3. The expectation

When we use mocks, we have to set the expectation before we execute the code. The words we use in RSpec help us to remember this, we are **expecting** our object **to receive** a message in the future. So the expectation needs to come before the code is executed.

Here's our test, broken down into its setup, execution and expectation, demonstrating how these steps are not occurring in the order we are used to.

{% highlight ruby %}
RSpec.describe 'Terminate' do
  it 'initiates self destruct' do

  #Setup
  fleep = Robot.new("Fleep")

  #Expectation
  expect(Terminate).to receive(:initiate).and_return("The test self destruct sequence has run")

  #Execution
  fleep.self_destruct
  end
end
{% endhighlight %}

If this disorder seems a bit upsetting then keep an eye out for the next post about **spies** which, spoiler alert, will help us put our tests back into the order we are used to.
