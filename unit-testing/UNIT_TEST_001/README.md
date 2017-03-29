###UNIT_TEST_001

#Unit Testing - 001

Testing Tools

In order to test our apps, we’ll use two tools: Jasmine and Karma.

##Jasmine

[Jasmine](http://jasmine.github.io/2.4/introduction.html) is a ***behavior-driven*** development framework for testing JavaScript code.

Using Jasmine, you can set expectations about what your code should do when invoked.

For instance, let’s assume we have a sum function on a Calculator object. We want to make sure that adding 1 and 1 results in 2. We could express that test (also called a _spec), by writing the following
code:

```javascript
describe('Calculator', () => {
  it('sums 1 and 1 to 2', () => {
    var calc = new Calculator();
    expect(calc.sum(1, 1)).toEqual(2);
  });
});
```

One of the nice things about Jasmine is how readable the tests are. You can see here that we expect the ```calc.sum``` operation to equal 2.

We organize our tests with ```describe``` blocks and ```it``` blocks.

Normally we use ```describe``` for each logical unit we’re testing and inside that each we use one ```it``` for each expectation you want to assert. However, this isn’t a hard and fast rule. You’ll often see an ```it``` block contain several expectations.

On the Calculator example above we have a very simple object. For that reason, we used one ```describe``` block for the whole class and one ```it``` block for each method.

This is not the case most of the times. For example, methods that produce different outcomes depending on the input will probably have more than one ```it``` block associated. On those cases, it’s perfectly fine to have ***nested describes***: one for the object and one for each method, and then different assertions inside individual ```it``` blocks.

##Karma

With Jasmine we can describe our tests and their expectations. Now, in order to actually run the tests we need to have a browser environment.

That’s where Karma comes in. Karma allows us to run JavaScript code within a browser like Chrome or Firefox, or on a headless browser (or a browser that doesn’t expose a user interface) like PhantomJS.

#Writing Unit Tests

Our main focus on this section will be to understand how we write unit tests against different parts of our Angular apps.

We’re going to learn to test Services, Components, HTTP requests and more. Along the way we’re also going to see a couple of different techniques to make our code more testable.


