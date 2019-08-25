---
layout: post
title: Test Spies
categories: [coding, code, learn to code, ruby, rspec, testing, unit test, double, dummy, stubs, mocks, spies]
---
#### A spy is an object that is set up to record the methods it receives during a test.

## Syntax
{% highlight ruby %}
spy_object = spy
expect(spy_object).to have_received(:message)
{% endhighlight %}

## Why would you use a spy?

There are two main reasons for using a spy.
1. To check whether an object has or has not received a certain method call while keeping the steps of our test in the conventional order.
2. To check whether any method has been received by an object without having to specify the particular method we are expecting.

### Test Order
As we discussed in our mocks article, a mock lets you set up the expectation that a certain method will or will not be called on an object. However, the syntax for a mock forces us to write the expectation section of our test before the execution phase which can reduce clarity and readability.

Here's our mock, broken down into its setup, execution and expectation, demonstrating how these steps are not occurring in the order we are used to.

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
{% endhighlight %}`
Spies can help us put our tests back in their conventional order. They allow us to setup our object to be a spy in the first stage of the test and then check to see whether it did or did not receive a certain method during the final expectation phase using the `have_received` syntax.

`{% highlight ruby %}
RSpec.describe 'Terminate' do
  it 'initiates self destruct' do
    #Setup
    stub_const("Terminate", spy)
    fleep = Robot.new('Fleep')

    #Execution
    fleep.self_destruct

    #Expectation
    expect(Terminate).to have_received(:initiate)
  end
end
{% endhighlight %}`

##### A note about `stub_const`
In this example we are using the stub_const method to change the value of an object (in this case a class) that should be unchanging because it's a constant. The syntax you will see more often for spies is what is documented in the syntax section.

##### A note about "Act, Arrange, Assert"
You may also have heard of or learnt about the test order steps by the mnemonic "AAA" which stands for:
- Arrange
- Act
- Assert

These refer to the same steps as setup, expectation and execution. While the alliteration is very pleasing I don't personally find those words meaningful or memorable and prefer not to use them.

### Receiving unexpected methods

But wait, there's more! There are also cases when writing tests when you may need to use a spy for a reason other than re-ordering tests.

Unlike other kind of test doubles, **you do not need to specify what methods you are expecting or not expecting the spy to receive**. Spies will simply record all method calls they receive.

This makes spies particularly suited to testing interactions with other pieces of code where you can't always predict what messages will be sent and received but you do want to check that communication is taking place. Examples of this might be API's that you don't control or testing logging functionality in your app.

## More options for spies
You can also expect a spy to have received a message with a particular argument, here are a few more examples of syntax you might see.

{% highlight ruby %}
expect(object).to have_received(:message).with(specific_argument)
{% endhighlight %}
Or, you can expect the message to be received a particular number of times.
{% highlight ruby %}
expect(object).to have_received(:message).twice
# or
expect(object).to have_received(:message).at_least(3).times
# or
expect(object).to have_received(:message).exactly(:once)

{% endhighlight %}
