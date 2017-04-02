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

***index_1.html***:

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
        Posted <a href="/2008/01/blah/57/" title="2008-01-28T20:24:17Z">January 28th, 2008</a>
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




