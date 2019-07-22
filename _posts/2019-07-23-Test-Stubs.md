---
layout: post
title: Test Stubs
categories: [coding, code, learn to code, ruby, rspec, testing, unit test, double, dummy, stubs]
---
#### A stub is an object we have set up to give us a predefined response when we call it.

## Syntax
Here are three different options for creating a test double and stubbing a method on it. The first is a little more readable, the second lets you return a block of code giving you more flexibility and the third is a shorthand giving us a more concise syntax.

{% highlight ruby %}
allow(object).to receive(:stubbed_method).and_return(return_value)
# OR
allow(object).to receive(:stubbed_method) { return_value }
#OR
object = double(stubbed_method: return_value)

expect(object.stubbed_method).to eq(your_expected_result)
{% endhighlight %}

We can read the first three examples of the stub setup as:
> Allow the object under test to receive the method we are stubbing and return the value we are defining.

The third example has the same meaning as the first two but is written in RSpec shorthand. You can use this shorthand to pass a hash of key-value pairs that represent many stubbed methods and their matching responses. Don't worry if this example doesn't make as much sense yet - we will discuss it in greater detail below.

We can read the expectation on the final line as:
> When the stubbed method is called on the object, we expect the return value we have redefined to be result.

## Why would you use a stub?

In general, methods should do one of two things:
1. query - return information or a response
2. command - change the state of the system

When we write unit tests for query methods we quickly run into a problem. We need our query method to return a piece of information but usually that information comes from another class or an external system and contacting that other system would break our unit test's isolation.

This is where stubs come in, stubs can let us set a pre-defined response for a method call so that we can check our method runs correctly while avoiding making calls to APIs, external databases or other systems that we want to avoid using in our unit tests.

## Code example
You can use stubs on both real and fake objects when testing. We are going to take a look at both options in our examples below.

Once again, here is our `Robot` class. We want to give our robot the ability to check the weather before it takes flight to ensure it doesn't take off into heavy rain or windy conditions.

{% highlight ruby %}
class Robot

  attr_reader :name, :flight_controller

  def initialize(name, flight_controller = FlightController.new)
    @name = name
    @flight_controller = flight_controller
  end

  def fly
    if flight_controller.good_weather?
      flight_controller.blast_off
    else
      puts "Sorry! I can't fly right now due to poor weather."
    end
  end
end
{% endhighlight %}

Above, we have added some code to our `fly` method that checks for `good_weather?` before blasting off.

{% highlight ruby %}
class FlightController

  attr_reader :weather

  def initialize(weather = Weather.new)
    @weather = weather
  end

  def good_weather?
    ['sunny', 'clear', 'overcast'].include?(weather.call_weather_api)
  end

  def blast_off
    "3 ... 2 ... 1 ... BLAST OFF!"
  end
end
{% endhighlight %}

In our `FlightController` class we have added a new instance variable `@weather` which has a default initializer of `Weather.new`. We also have a method `good_weather?` that calls `call_weather_api` on the weather object. It checks if the external weather service has reported sunny, clear or overcast weather, if it has, it will return true, and if not, it will return false.

{% highlight ruby %}
class Weather
  def call_weather_api
    puts "This calls an external API"
    ["windy", "sunny", "snowy", "raining", "stormy", "extreme heat", "clear", "overcast"].sample
  end
end
{% endhighlight %}

Finally, we have defined a new `Weather` class which is responsible for the API call to the external weather service. To represent this in an extremely simplified way for our example, we've got a list of possible weather conditions which are randomly selected and returned with the `.sample` method.

## Test examples

For this example, we are going to create a unit test for the `FlightController's` method `good_weather?`. We're going to look at **three different ways** of writing this test.

{% highlight ruby %}
def good_weather?
  ['sunny', 'clear', 'overcast'].include?(weather.call_weather_api)
end
{% endhighlight %}

As we've discussed previously, we want to isolate this unit test from its dependency on the `Weather` class and the external API call that occurs in the `call_weather_api` method.

To write our tests, we will stub out the `call_weather_api method`. Below, we have created a new weather object called `tomorrows_weather` and are allowing it to receive the `.call_weather_api` method and requiring it to return the string `"stormy"`.

{% highlight ruby %}
RSpec.describe 'FlightController' do
  describe '#good_weather?' do
    it "returns false when weather is stormy" do
      tomorrows_weather = Weather.new()
      allow(tomorrows_weather).to receive(:call_weather_api).and_return("stormy")
      expect(FlightController.new(tomorrows_weather).good_weather?).to eq(false)
    end
  end
end
{% endhighlight %}
Now, you may have noticed that while we create `tomorrows_weather` the only method we call on it is a stub so it's a bit of a waste to create a whole weather object and not use any of its real functionality. It is also a little odd to create a new `Weather` object in a test for `FlightController` which we are attempting to isolate from other classes. While we have stubbed out the method call to `call_weather_api` we still have a dependency on the `Weather` class.

If we don't want to initialize a whole new weather object for our test we can instead stub the method on a test double. The code will look very familiar to those who remember the post on **dummy** objects but this isn't a true dummy object because we are expecting it to receive a message.

{% highlight ruby %}
RSpec.describe 'FlightController' do
  describe '#good_weather?' do
    it "returns true when weather is sunny" do
      todays_weather = double
      allow(todays_weather).to receive(:call_weather_api).and_return("sunny")
      expect(FlightController.new(todays_weather).good_weather?).to eq(true)
    end
  end
end
{% endhighlight %}
This process of stubbing a method on a test double is a very common thing to do in RSpec tests and because of this we have a short-hand for it. When we create our `mondays_weather` test double we can pass in a hash to set a number of methods for the double to receive with matching responses.

{% highlight ruby %}
RSpec.describe 'FlightController' do
  describe '#good_weather?' do
    it "returns true when weather is sunny" do
      mondays_weather = double({ call_weather_api: 'sunny', second_stubbed_method: true })
      expect(FlightController.new(mondays_weather).good_weather?).to eq(true)
    end
  end
end
{% endhighlight %}
