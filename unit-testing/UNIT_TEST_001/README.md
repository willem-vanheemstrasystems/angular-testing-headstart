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

##HTTP Considerations

We could start writing our tests right now, but during each test execution we would be calling out and hitting the Spotify server. This is far from ideal for three reasons:

1. HTTP requests are relatively slow and as our test suite grows, we’d notice it takes longer and longer to run all of the tests.

2. Spotify’s API has a quota, and if our whole team is running the tests, we might use up our API call resources needlessly

3. If we are offline or if Spotify is down or inaccessible our tests would start breaking, even though our code might technically be correct

This is a good hint when writing unit tests: ***isolate everything that you don’t control before testing***.

In our case, this piece is the Spotify service. The solution is that we will replace the HTTP request with something that would behave like it, but will not hit the real Spotify server.

Doing this in the testing world is called ```mocking``` a dependency. They are sometimes also called ```stubbing``` a dependency.

NOTE: You can read more about the difference between Mocks and Stubs in this article [Mocks are not Stubs](http://martinfowler.com/articles/mocksArentStubs.html).

Let’s pretend we’re writing code that depends on a given ```Car``` class. This class has a bunch of methods: you can ```start``` a car instance, ```stop``` it, ```park``` it and ```getSpeed``` of that car. Let’s see how we could use stubs and mocks to write tests that depend on this class.

##Stubs

Stubs are objects we create on the fly, with a subset of the behaviors our dependency has.
Let’s write a test that just interacts with the ```start``` method of the class.
You could create a stub of that ```Car``` class on-the-fly and inject that into the class you’re testing:

```javascript
describe('Speedtrap', function() {
  it('tickets a car at more than 60mph', function() {
    var stubCar = { getSpeed: function() { return 61; } };
    var speedTrap = new SpeedTrap(stubCar);
    speedTrap.ticketCount = 0;
    speedTrap.checkSpeed();
    expect(speedTrap.ticketCount).toEqual(1);
  });
});
```

This would be a typical case for using a stub and we’d probably only use it locally to that test.

##Mocks

Mocks in our case will be a more complete representation of objects, that overrides parts or all of the behavior of the dependency. Mocks can, and most of the time will be reused by more than one test across our suite.

They will also be used sometimes to assert that given methods were called the way they were supposed to be called.

One example of a mock version of our ```Car``` class would be:

```javascript
class MockCar {
  startCallCount: number = 0;

  start() {
    this.startCallCount++;
  }
}
```

And it would be used to write another test like this:

```javascript
describe('CarRemote', function() {
  it('starts the car when the start key is held', function() {
    var car = new MockCar();
    var remote = new CarRemote();
    remote.holdButton('start');
    expect(car.startCallCount).toEqual(1);
  });
});
```

The biggest difference between a ```mock``` and a ```stub``` is that:

• a ***stub*** provides a subset of functionality with “manual” behavior overrides, whereas

• a ***mock*** generally sets expectations and verifies that certain methods were called.

##Http MockBackend

Now that we have this background in mind, let’s go back to writing our service test code.

Interacting with the live Spotify service every time we run our tests is a poor idea but thankfully Angular provides us with a way to create ***fake HTTP calls*** with ```MockBackend```.

This class can be injected into an ```Http``` instance and gives us control of how we want the HTTP interaction to act. We can interfere and assert in a variety of different ways: we can manually set a response, simulate an HTTP error, and add expectations, like asserting the URL being requested matches what we want, if the provided request parameters are correct and a lot more.

So the idea here is that we’re going to provide our code with a “fake” ```Http``` library. This “fake” library will appear to our code to be the real ```Http``` library: all of the methods will match, it will return responses and so on. However, we’re not actually going to make the requests.

In fact, beyond not making the requests, our ```MockBackend``` will actually allow us to setup expectations and watch for behaviors we expect.

##TestBed.configureTestingModule and Providers

When we test our Angular apps we need to make sure we configure the top-level ```NgModule``` that we will use for this test. When we do this, we can configure providers, declare components, and import other modules: just like you would when using NgModules generally.

Sometimes when testing Angular code, we ```manually setup injections```. This is good because it gives us more control over what we’re actually testing.

So in the case of testing ```Http``` requests, we don’t want to inject the “real” ```Http``` class, but instead we want to inject something that looks like ```Http```, but really intercepts the requests and returns the responses we configure.

To do that, we create a version of the ```Http``` class that uses ```MockBackend``` internally.

To do this, we use the ```TestBed.configureTestingModule``` in the ```beforeEach``` hook. This ```hook``` takes a callback function that will be called before each test is run, giving us a great opportunity to configure alternative class implementations.

-- See music/src/app/spotify.service.spec.ts --

```javascript
describe('SpotifyService', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        BaseRequestOptions,
        MockBackend,
        SpotifyService,
        { provide: Http,
          useFactory: (backend: ConnectionBackend,
                       defaultOptions: BaseRequestOptions) => {
                         return new Http(backend, defaultOptions);
                       }, deps: [MockBackend, BaseRequestOptions] },
      ]
    });
  });
```

Notice that ```TestBed.configureTestingModule``` accepts an array of providers in the ```providers``` key to be used by the test injector.

```BaseRequestOptions``` and ```SpotifyService``` are just the default implementation of those classes. But the last provider is a little more complicated:

```javascript
        { provide: Http,
          useFactory: (backend: ConnectionBackend,
                       defaultOptions: BaseRequestOptions) => {
                         return new Http(backend, defaultOptions);
                       }, deps: [MockBackend, BaseRequestOptions] }
```

This code uses ```provide``` with ```useFactory``` to create a version of the ```Http``` class, using a factory (that’s what useFactory does).

That factory has a signature that expects ```ConnectionBackend``` and ```BaseRequestOption``` instances.

The second key on that object is ```deps: [MockBackend, BaseRequestOptions]```. That indicates that we’ll be using ```MockBackend``` as the first parameter of the factory and ```BaseRequestOptions``` (the default implementation) as the second.

Finally, we return our customized ```Http``` class with the ```MockBackend``` as a result of that function.

What benefit do we get from this? Well now every time (in our test) that our code requests ```Http``` as an injection, it will instead receive our customized ```Http``` instance.

This is a powerful idea that we’ll use a lot in testing: use dependency injection to customize dependencies and isolate the functionality you’re trying to test.

##Testing getTrack

Now, when writing tests for the service, we want to verify that we’re calling the correct URL.

Let’s write a test for the ```getTrack``` method of music/src/app/spotify.service.ts:

```javascript
  getTrack(id: string): Observable<any[]> {
    return this.query(`/tracks/${id}`);
  }
```

If you remember how that method works, it uses the query method, that builds the URL based on the parameters it receives:

```javascript
  query(URL: string, params?: Array<string>): Observable<any[]> {
    let queryURL = `${SpotifyService.BASE_URL}${URL}`;
    if (params) {
      queryURL = `${queryURL}?${params.join('&')}`;
    }

    return this.http.request(queryURL).map((res: any) => res.json());
  }
```

Since we’re passing ```/tracks/${id}``` we assume that when calling ```getTrack('TRACK_ID')``` the expected URL will be ```https://api.spotify.com/v1/tracks/TRACK_ID```.

Here is how we write the test for this:

```javascript
  describe('getTrack', () => {
    it('retrieves using the track ID',
      inject([SpotifyService, MockBackend], fakeAsync((svc, backend) => {
        let res;
        expectURL(backend, 'https://api.spotify.com/v1/tracks/TRACK_ID');
        svc.getTrack('TRACK_ID').subscribe((_res) => {
          res = _res;
        });
        tick();
        expect(res.name).toBe('felipe');
      }))
    );
  });
```





