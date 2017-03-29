# angular-testing-headstart
Angular Testing - Headstart

Based on 'ng Book 2 Angular 4: Testing'

#End-to-end Testing (e2e-testing)

If you take a top-down approach on testing you write tests that see the application as a “black box”
and you interact with the application like a user would and evaluate if the app seems to work from
the “outside”. This top-down technique of testing is called End to End testing.

NOTE: In the Angular world, the tool that is mostly used is called ***Protractor***. Protractor is a
tool that opens a browser and interacts with the application, collecting results, to check
whether the testing expectations were met.

#Unit Testing (unit-testing)

The second testing approach commonly used is to isolate each part of the application and test it in
isolation. This form of testing is called Unit Testing.

In Unit Testing we write tests that provide a given input to a given aspect of that unit and evaluate
the output to make sure it matches our expectations.
