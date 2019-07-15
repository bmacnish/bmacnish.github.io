---
layout: post
title: Test Dummies
categories: [coding, code, learn to code, ruby, rspec, testing, unit test, double, dummy]
---
#### A dummy is a basic placeholder test double that doesn't respond to any messages. We use dummies when we want to pass an object into a method as an argument but don't need any of its functionality in our test. Dummies are the most straight forward kind of test double.

## Syntax
The syntax for creating a dummy object in RSpec is very simple. You set a variable name to the value of `double`. `double` is a special kind of RSpec object with some fancy functionality that we will get into a bit further down the track. For the moment through, we are using it as a blank (or dummy) object.

{% highlight ruby %}
dummy = double
{% endhighlight %}
## Example code
To learn about dummies we are going to dive straight into an example. Here we have some basic code defining our Robot class.

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
{% endhighlight %}
## Example test
We have decided that we want to test the ability to read the `name` property in the `initialize` method on our `Robot` class.

In the test below, we want to initialize a new robot, give it the `name` Fleep and check that this name has sucessfully been assigned.

{% highlight ruby %}
RSpec.describe 'robot' do
  describe '.new' do
    it 'allows reading of name' do
      expect(Robot.new("Fleep").name).to eql("Fleep")
    end
  end
end
{% endhighlight %}

However, we run into trouble when we try and run this test. Our `Robot` class initializer takes a second argument `flight_controller` and while this argument has a default value of object `FlightController.new` we haven't yet implimented the `FlightController` class and don't have any way to pass an object of this type.

When we run the code we have so far, we get the following error.

{% highlight ruby %}
     NameError:
       uninitialized constant Robot::FlightController
{% endhighlight %}

One way to solve this problem is to use a test double.

We will pass a **dummy** object using the keyword `double` which will allow us to test the functionality of the `name` method without worrying whether `FlightController` is yet implemented. We could also choose to use a dummy object even if we had the `FlightController` code as it would let us run our test without generating an unnecessary `FlightController` object that would be created but then not used.

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


RSpec.describe 'robot' do
  describe '.new' do
    it 'allows reading of name' do
      flight_controller_dummy = double

      expect(Robot.new("Fleep", flight_controller_dummy).name).to eql("Fleep")
    end
  end
end
{% endhighlight %}
In the test above, we can see that a new object `flight_controller_dummy` has been created and is a double object. We can then pass this double to our new `Robot` class as the second argument and the test will pass.

Here we have used a simple dummy object as a placeholder for a `FlightController` object letting us isolate the testing of `@name`. However, this is not all that RSpec's `double` object can do and we will look into more sophisticated usages as we move onto **stubs, mocks and spies**.
