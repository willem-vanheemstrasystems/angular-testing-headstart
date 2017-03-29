###UNIT_TEST_001

#Unit Testing - 001

Testing Tools

In order to test our apps, we’ll use two tools: Jasmine and Karma.

##Jasmine

Jasmine is a ***behavior-driven*** development framework for testing JavaScript code.
Using Jasmine, you can set expectations about what your code should do when invoked.
For instance, let’s assume we have a sum function on a Calculator object. We want to make sure that
adding 1 and 1 results in 2. We could express that test (also called a _spec), by writing the following
code:

```javascript
describe('Calculator', () => {
  it('sums 1 and 1 to 2', () => {
    var calc = new Calculator();
    expect(calc.sum(1, 1)).toEqual(2);
  });
});
```



##Karma