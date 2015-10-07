# Creating Ajax calls with JavaScript

_A quick guide to create a simple ajax call in JavaScript. As well as an example of a fluent interfaces object_

**Published:**  2013/12/06 (December 6 2013)

So recently I was trying to work with raw JavaScript to create ajax calls. I was surprised to find such poorly written articles about the topic and how many people just suggested using jQuery instead of learning how to do it the raw way. In this tutorial I'll show and explain each part of the ajax process.

## Wait, what is ajax?!
Ajax stands for Asynchronous JavaScript and XML. Ajax allows you to retrieve or post a page's content without actually having to load the page in the browser window. This is great if you want to create real time applications that retrieve, update, validate, etc... with out needing to refresh the page.

## So how do we make an Ajax call?
First we need to create a XMLHttpRequest object. In this case we will be storing it in a variable called xmlhttp.

```javascript
xmlhttp = new XMLHttpRequest();
```

The above will get use started however it will only work in IE8+, Firefox, Chrome, Opera, Safari. If we want to support older versions of IE (6, 7) then we need to use ActiveX like so.

```javascript
xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
```

In order to have both of these in our code we will create an if statement.

```javascript
if(window.XMLHttpRequest) {
    xmlhttp = new XMLHttpRequest();
} else {
    xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
}
```

Next we need to open the path to the file, this is done by using open().

```javascript
xmlhttp.open(method, url, async);
```

The open function takes three parameters

**method**
> Either GET or POST

**url**
> The path to the file to be opened, if we use GET above we will pass the variables inside this function, otherwise use the .send() function to pass variables with POST

**async**
> true (asynchronous) or false (synchronous)

By default we should use true as it produces consisten results, false will just load the page and not check if it exist. Many times it will not even load the page.

If we need to specify any headers for the page we are loading we should do that with the `setRequestHeader()` function:

```javascript
xmlhttp.setRequestHeader();
```

Example to return a page with plain text

```javascript
xmlhttp.setRequestHeader("Content-type", "text/plain");
```

Now we call the send() function and pass any variables if we are using POST

```javascript
xmlhttp.send();
```

Example for POST

```javascript
xmlhttp.send("id=about&sub=contact");
```

To return our page data we have two options. If we set the async above to false you can return the page by just using this:

```javascript
document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
```

In the above example we set the response text (`xmlhttp.responseText`) into the contents of the container called "myDiv".

If we set the async to true we use the following code.

```javascript
xmlhttp.onreadystatechange=function() {
    if(xmlhttp.readyState == 4 && xmlhttp.status == 200) {
        document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
    }
}
```

We continue to check the readyState of the ajax call and if it is equal to 4 and the status is 200 we return the responseText like about into a container called "myDiv"

## The finished product
Below is a basic example using GET, going to page example.html with async set to true:

```javascript
// Create XMLHttpRequest Object
if(window.XMLHttpRequest) {
    // code for IE7+, Firefox, Chrome, Opera, Safari
    xmlhttp = new XMLHttpRequest();
} else {
    // code for IE6, IE5
    xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
}
// Open path
xmlhttp.open("GET", "example.html", true);
// Send any data if needed
xmlhttp.send();
// Check if we are ready to return the contents
xmlhttp.onreadystatechange = function() {
    // If request finished and response is ready
    if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
        // Return conent of the page
        document.getElementById("myDiv").innerHTML = xmlhttp.responseText;
    }
}
```

## What is readyState & status?
readyState & status (spelled exactlly like that) is the current status of the page requested. readyState has 5 states, which are:

**0**
> Request not initialized

**1**
> Server connection established

**2**
> Request received

**3**
> Processing request

**4**
> Request finished and response is ready (What we want)

** Status on the other hand can only return 2 responses: **

**200**
> "OK" (What we want)

**404**
> Page not found 

`onreadystatechange()` will continue to check the status of the request and once it hits 4 for the readyState and 200 for the stauts the page will return the contents.

I'm using the above code but it's not work!

I ran into the problem that I was getting a 0 on readyState and a 200 on status. Even though my code was right the problem was I was not on a server. The function `.open()` only allows path's with http or https. Since you file system (if you are windows) uses either file:// or c:/ you will find that the code does not work.

Solution; Either run the code on a local server with lamp, wamp, xamp, mamp, etc.. or run it on a remote server. As long as the path is http or https it should work!

## Seems messy, anything to clean it up?
I decided to create a Fluent Interfaces ajax object that does the above but a bit cleaner.

```javascript
var ajax = {
    container: function (data) {
        ajax._container = (data != undefined) ? data : undefined;
        return ajax;
    },
    url: function (data) {
        ajax._url = (data != undefined) ? data : undefined;
        return ajax;
    },
    method: function (data) {
        ajax._method = (data != undefined) ? data : "GET";
        return ajax;
    },
    send: function (data) {
        ajax._send = (data != undefined) ? data : undefined;
        return ajax;
    },
    async: function (data) {
        ajax._async = (data != undefined) ? data : true;
        return ajax;
    },
    cache: function (data) {
        ajax._cache = (data != undefined) ? data : false;
        return ajax;
    },
    headers: function (data) {
        ajax._headers = (data != undefined) ? data : undefined;
        return ajax;
    },
    render: function () {
        // Check for needed variables
        if(ajax._container === undefined || ajax._url === undefined) {
            console.log("Please define a container and url");
            return false;
        }
        // Check for other variables
        ajax._method  = (ajax._method != undefined) ? ajax._method : "GET";
        ajax._send    = (ajax._send != undefined) ? ajax._send : undefined;
        ajax._async   = (ajax._async != undefined) ? ajax._async : true;
        ajax._cache   = (ajax._cache != undefined) ? ajax._cache : false;
        ajax._headers = (ajax._headers != undefined) ? ajax._headers : undefined;
        // Create XMLHttpRequest Object
        if (window.XMLHttpRequest) {
            // code for IE7+, Firefox, Chrome, Opera, Safari
            xmlhttp = new XMLHttpRequest();
        } else {
            // code for IE6, IE5
            xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");
        }
        // Cache?
        if(ajax._cache == false) {
            ajax._ajaxCache = "?ajaxCache=" + Math.random();
        } else {
            ajax._ajaxCache = '';
        }
        // Open path
        xmlhttp.open(ajax._method, ajax._url + ajax._ajaxCache, ajax._async);
        //Headers
        if(ajax._headers != undefined) {
            xmlhttp.setRequestHeader(ajax._headers);
        }
        // Send any data if needed
        xmlhttp.send(ajax._send);
        // Use Async?
        if(ajax._async == true) {
            // Check if we are ready to return the contents
            xmlhttp.onreadystatechange=function() {
                // If request finished and response is ready
                if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
                    // Return the variable "content"
                    document.getElementById(ajax._container).innerHTML = xmlhttp.responseText;
                }
            }
        } else {
            // Return the variable "content"
            document.getElementById(ajax._container).innerHTML = xmlhttp.responseText;
        }
        return ajax;
    }
};
```

Here is an example to call it into the container "content" from the url example.html

```javascript
ajax.container('content').url("example.html").render();
```

You can either pass it with an onlick parameter

```html
<button onclick="ajax.container('content').url('example.html').render();">load page</button>
```

If you want to make every a href an ajax call you could loop it with this:

```javascript
document.addEventListener("DOMContentLoaded", function() {
    // get all dom elements with class "feature"
    var aFeatures = document.querySelectorAll("a");
    // for each selected element
    for (var i = 0; i < aFeatures.length; i++) {
        // add click handler
        aFeatures[i].addEventListener('click', function (event) {
            // Ajax Object
            ajax.container('content')
                .url(this.href)
                .render();
            event.preventDefault();
            return false;
        });
    }
});
```

It does not have to be a href call, you could make it a class instead.
