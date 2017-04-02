###README.md

Based on 'Introduction To JavaScript Unit Testing' at https://www.smashingmagazine.com/2012/06/introduction-to-javascript-unit-testing/

#UNIT_TEST_002

You probably know that testing is good, but the first hurdle to overcome when trying to write unit tests for client-side code is the lack of any actual units; JavaScript code is written for each page of a website or each module of an application and is closely intermixed with back-end logic and related HTML. In the worst case, the code is completely mixed with HTML, as inline events handlers.

This is likely the case when no JavaScript library for some DOM abstraction is being used; writing inline event handlers is much easier than using the DOM APIs to bind those events. More and more developers are picking up a library such as jQuery to handle the DOM abstraction, allowing them to move those inline events to distinct scripts, either on the same page or even in a separate JavaScript file. However, putting the code into separate files doesn’t mean that it is ready to be tested as a unit.

What is a unit anyway? In the best case, it is a pure function that you can deal with in some way — a function that always gives you the same result for a given input. This makes unit testing pretty easy, but most of the time you need to deal with side effects, which here means DOM manipulations. It’s still useful to figure out which units we can structure our code into and to build unit tests accordingly.

#Building Unit Tests

With that in mind, we can obviously say that starting with unit testing is much easier when starting something from scratch. But that’s not what this article is about. This article is to help you with the harder problem: extracting existing code and testing the important parts, potentially uncovering and fixing bugs in the code.

The process of extracting code and putting it into a different form, without modifying its current behavior, is called refactoring. Refactoring is an excellent method of improving the code design of a program; and because any change could actually modify the behaviour of the program, it is safest to do when unit tests are in place.

This chicken-and-egg problem means that to add tests to existing code, you have to take the risk of breaking things. So, until you have solid coverage with unit tests, you need to continue manually testing to minimize that risk.

That should be enough theory for now. Let’s look at a practical example, testing some JavaScript code that is currently mixed in with and connected to a page. The code looks for links with ```title``` attributes, using those titles to display when something was posted, as a relative time value, like “5 days ago”:

***0_mangled.html***:

```javascript
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Mangled date examples</title>
    <script>
    function prettyDate(time){
        var date = new Date(time || ""),
            diff = ((new Date().getTime() - date.getTime()) / 1000),
            day_diff = Math.floor(diff / 86400);

        if (isNaN(day_diff) || day_diff < 0 || day_diff >= 31) {
            return;
        }

        return day_diff == 0 && (
                diff < 60 && "just now" ||
                diff < 120 && "1 minute ago" ||
                diff < 3600 && Math.floor( diff / 60 ) + " minutes ago" ||
                diff < 7200 && "1 hour ago" ||
                diff < 86400 && Math.floor( diff / 3600 ) + " hours ago") ||
            day_diff == 1 && "Yesterday" ||
            day_diff < 7 && day_diff + " days ago" ||
            day_diff < 31 && Math.ceil( day_diff / 7 ) + " weeks ago";
    }
    window.onload = function(){
        var links = document.getElementsByTagName("a");
        for (var i = 0; i < links.length; i++) {
            if (links[i].title) {
                var date = prettyDate(links[i].title);
                if (date) {
                    links[i].innerHTML = date;
                }
            }
        }
    };
    </script>
</head>
<body>

<ul>
<li class="entry" id="post57">
    <p>blah blah blah…</p>
    <small class="extra">
        Posted <a href="/2017/01/blah/57/" title="2017-01-01T20:24:17Z">January 1st, 2017</a>
        by <a href="/john/">John Resig</a>
    </small>
</li>
<!-- more list items -->
</ul>

</body>
</html>
```

If you ran that example, you’d see a problem: ***none of the dates get replaced***. The code works, though. It loops through all anchors on the page and checks for a title property on each. If there is one, it passes it to the prettyDate function. If prettyDate returns a result, it updates the innerHTML of the link with the result.

#Make Things Testable 

The problem is that for any date older than 31 days, ```prettyDate``` just returns ***undefined*** (implicitly, with a single ```return``` statement), leaving the text of the anchor as is. So, to see what’s supposed to happen, we can hardcode a “current” date:

NOTE: You have to change the date fields in below file to be within 31 days of now.

***1_mangled.html***:

```javascript
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Mangled date examples</title>
    <script>
    function prettyDate(now, time){
        var date = new Date(time || ""),
            diff = (((new Date(now)).getTime() - date.getTime()) / 1000),
            day_diff = Math.floor(diff / 86400);

        if (isNaN(day_diff) || day_diff < 0 || day_diff >= 31) {
            return;
        }

        return day_diff == 0 && (
                diff < 60 && "just now" ||
                diff < 120 && "1 minute ago" ||
                diff < 3600 && Math.floor( diff / 60 ) + " minutes ago" ||
                diff < 7200 && "1 hour ago" ||
                diff < 86400 && Math.floor( diff / 3600 ) + " hours ago") ||
            day_diff == 1 && "Yesterday" ||
            day_diff < 7 && day_diff + " days ago" ||
            day_diff < 31 && Math.ceil( day_diff / 7 ) + " weeks ago";
    }
    window.onload = function(){
        var links = document.getElementsByTagName("a");
        for (var i = 0; i < links.length; i++) {
            if (links[i].title) {
                var date = prettyDate("2017-04-01T22:25:00Z", links[i].title);
                if (date) {
                    links[i].innerHTML = date;
                }
            }
        }
    };
    </script>
</head>
<body>

<ul>
<li class="entry" id="post57">
    <p>blah blah blah…</p>
    <small class="extra">
        Posted <a href="/2017/04/blah/57/" title="2017-04-01T20:24:17Z">April 1st, 2017</a>
        by <a href="/john/">John Resig</a>
    </small>
</li>
<!-- more list items -->
</ul>

</body>
</html>
```

Now, the links should say “2 hours ago,” “Yesterday” and so on. That’s something, but still not an actual testable unit. So, without changing the code further, all we can do is try to test the resulting DOM changes. Even if that did work, any small change to the markup would likely break the test, resulting in a really bad cost-benefit ratio for a test like that.

#Refactoring, Stage 0

Instead, let’s refactor the code just enough to have something that we can unit test.

We need to make two changes for this to happen: pass the current date to the ```prettyDate``` function as an argument, instead of having it just use ```new Date```, and extract the function to a separate file so that we can include the code on a separate page for unit tests.

***2_mangled.html***

```javascript
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Refactored date examples</title>
    <script src="prettydate.js"></script>
    <script>
    window.onload = function() {
        var links = document.getElementsByTagName("a");
        for ( var i = 0; i < links.length; i++ ) {
            if (links[i].title) {
                var date = prettyDate("2017-04-01T22:25:00Z", links[i].title);
                if (date) {
                    links[i].innerHTML = date;
                }
            }
        }
    };
    </script>
</head>
<body>

<ul>
<li class="entry" id="post57">
    <p>blah blah blah…</p>
    <small class="extra">
        Posted <a href="/2017/04/blah/57/" title="2017-04-01T20:24:17Z">April 1st, 2017</a>
        by <a href="/john/">John Resig</a>
    </small>
</li>
<!-- more list items -->
</ul>

</body>
</html>
```

***prettydate.js***

```javascript
function prettyDate(now, time){
    var date = new Date(time || ""),
        diff = (((new Date(now)).getTime() - date.getTime()) / 1000),
        day_diff = Math.floor(diff / 86400);

    if (isNaN(day_diff) || day_diff < 0 || day_diff >= 31) {
        return;
    }

    return day_diff == 0 && (
            diff < 60 && "just now" ||
            diff < 120 && "1 minute ago" ||
            diff < 3600 && Math.floor( diff / 60 ) + " minutes ago" ||
            diff < 7200 && "1 hour ago" ||
            diff < 86400 && Math.floor( diff / 3600 ) + " hours ago") ||
        day_diff == 1 && "Yesterday" ||
        day_diff < 7 && day_diff + " days ago" ||
        day_diff < 31 && Math.ceil( day_diff / 7 ) + " weeks ago";
}
```

Now that we have something to test, let’s write some actual unit tests:

***0_unit_test.html***:

```javascript
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Refactored date examples</title>
    <script src="prettydate.js"></script>
    <script>
    function test(then, expected) {
        results.total++;
        var result = prettyDate("2017-04-01T15:36:00Z", then);
        if (result !== expected) {
            results.bad++;
            console.log("Expected " + expected + ", but was " + result);
        }
    }
    var results = {
        total: 0,
        bad: 0
    };
    test("2017/04/01 15:36:00", "just now");
    test("2017/04/01 15:35:30", "1 minute ago");
    test("2017/04/01 14:36:30", "1 hour ago");
    test("2017/03/31 15:36:30", "Yesterday");
    test("2017/03/30 15:36:30", "2 days ago");
    test("2000/04/26 22:23:30", undefined);
    console.log("Of " + results.total + " tests, " + results.bad + " failed, "
        + (results.total - results.bad) + " passed.");
    </script>
</head>
<body>

</body>
</html>
```

NOTE: Make sure to enable a console such as Firebug or Chrome’s Web Inspector. The output will be displayed in the console, not on the web page.

This will create an ad-hoc testing framework, using only the console for output. It has no dependencies to the DOM at all, so you could just as well run it in a non-browser JavaScript environment, such as Node.js or Rhino, by extracting the code in the ```script``` tag to its own file.

If a test fails, it will output the expected and actual result for that test. In the end, it will output a test summary with the total, failed and passed number of tests.

If all tests have passed, like they should here, you would see the following in the console:

```javascript
Of 6 tests, 0 failed, 6 passed.
```

To see what a failed assertion looks like, we can change something to break it:

```javascript
Expected 2 day ago, but was 2 days ago.
Of 6 tests, 1 failed, 5 passed.
```

While this ad-hoc approach is interesting as a proof of concept (you really can write a test runner in just a few lines of code), it’s much more practical to use an existing unit testing framework that provides better output and more infrastructure for writing and organizing tests.

