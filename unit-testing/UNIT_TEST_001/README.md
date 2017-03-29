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

##Angular Unit Testing framework

Angular provides its own set of classes that build upon the Jasmine framework to help writing unit testing for the framework.

The main testing framework can be found on the ```@angular/core/testing``` package. (Although, for testing components we’ll use the ```@angular/compiler/testing``` package and ```@angular/platformbrowser/testing``` for some other helpers. But more on that later.)

##Setting Up Testing

Earlier in the Routing Chapter we created an application for searching for music. In this chapter, let’s write tests for that application.

Karma requires a configuration in order to run. So the first thing we need to do to setup Karma is to create a ```karma.conf.js``` file.

Let’s create a karma.conf.js file on the root path of our project, like so:

```javascript
// Karma configuration file, see link for more information
// https://karma-runner.github.io/0.13/config/configuration-file.html

module.exports = function (config) {
  config.set({
    basePath: '',
    frameworks: ['jasmine', '@angular/cli'],
    plugins: [
      require('karma-jasmine'),
      require('karma-chrome-launcher'),
      require('karma-jasmine-html-reporter'),
      require('karma-coverage-istanbul-reporter'),
      require('@angular/cli/plugins/karma')
    ],
    client:{
      clearContext: false // leave Jasmine Spec Runner output visible in browser
    },
    files: [
      { pattern: './src/test.ts', watched: false }
    ],
    preprocessors: {
      './src/test.ts': ['@angular/cli']
    },
    mime: {
      'text/x-typescript': ['ts','tsx']
    },
    coverageIstanbulReporter: {
      reports: [ 'html', 'lcovonly' ],
      fixWebpackSourcePaths: true
    },
    angularCli: {
      environment: 'dev'
    },
    reporters: config.angularCli && config.angularCli.codeCoverage
              ? ['progress', 'coverage-istanbul']
              : ['progress', 'kjhtml'],
    port: 9876,
    colors: true,
    logLevel: config.LOG_INFO,
    autoWatch: true,
    browsers: ['Chrome'],
    singleRun: false
  });
};
```

Since we’re using Angular CLI, this ```music/karma.conf.js``` file is already created for us! However, if your project does not use Angular CLI, you may need to setup Karma on your own.

Don’t worry too much about this file’s contents right now, just keep in mind a few things about it:

• sets PhantomJS as the target testing browser;

• uses Jasmine karma framework for testing;

• uses a WebPack bundle called ```test.bundle.js``` that basically wraps all our testing and app code;

The next step is to create a new ```test``` folder to hold our test files.

```javascript
cd music
mkdir test
```

##Testing Services and HTTP

Services in Angular start out their life as plain classes. In one sense, this makes our services easy to test because we can sometimes test them directly without using Angular.

With Karma configuration done, let’s start testing the ```SpotifyService``` class. If we remember, this service works by interacting with the Spotify API to retrieve album, track and artist information.
Inside the test folder where all our service tests will go. Finally, let’s create our first test file inside the app folder, called ```spotify.service.spec.ts```.
Now we can start putting this test file together. The first thing we need to do is import the test helpers from the ```@angular/core/testing``` package:

-- See music/src/app/spotify.service.spec.ts --

```javascript
import {
  inject,
  fakeAsync,
  tick,
  TestBed
} from '@angular/core/testing';
```

Next, we’ll import a couple more classes:

```javascript
import {MockBackend} from '@angular/http/testing';
import {
  Http,
  ConnectionBackend,
  BaseRequestOptions,
  Response,
  ResponseOptions
} from '@angular/http';
```

Since our service uses HTTP requests, we’ll import the ```MockBackend``` class from ```@angular/http/testing``` package. This class will help us set expectations and verify HTTP requests.

The last thing we need to import is the class we’re testing:

```javascript
import { SpotifyService } from './spotify.service';
```

