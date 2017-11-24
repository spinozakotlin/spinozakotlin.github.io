---
layout: default
title: Documentation
rightmenu: true
permalink: /documentation
---

<div id="spy-nav" class="right-menu" markdown="1">
* [Getting Started](#getting-started)
* [Stopping the Server](#stopping-the-server)
* [Routes](#routes)
* [Route context (request/response)](#implicitly-available-functionality-in-route-context)
* [Sessions](#sessions)
* [Halting](#halting)
* [Filters](#filters)
* [Redirects](#redirects)
* [Error Handling](#error-handling)
* [Exception Mapping](#exception-mapping)
* [Configuration DSL](#configuration-dsl)
* [Embedded Webserver](#embedded-web-server)
  * [- Port](#port)
  * [- Secure](#secure)
  * [- ThreadPool](#threadpool)
  * [- Await initialization](#awaitinit)
  * [- WebSockets](#websockets)
* [Other Webserver](#other-web-server)
* [GZIP](#gzip)
* [Examples/FAQ](#examples-and-faq)
</div>

<h1 class="no-margin-top">Documentation</h1>
Documentation here is always for the latest version of Spark. We don't have the capacity to maintain separate docs for each version, but Spark is always backwards compatible.

Docs for [(spark-kotlin)](http://sparkjava.com/news#spark-kotlin-released) will arrive here ASAP.
You can follow the progress of spark-kotlin on [(GitHub)](https://github.com/perwendel/spark-kotlin)       

<div class="star-one star-two">
    <div>
        If you like spark-kotlin star us and help the community grow:
    </div>
    <iframe id="starFrame" class="githubStar"
            src="https://ghbtns.com/github-btn.html?user=perwendel&amp;repo=spark-kotlin&amp;type=star&amp;count=true&size=large"
            frameborder="0" scrolling="0" width="205px" height="30px">
    </iframe>
</div>

## Getting started

**1:** Create a new gradle project and add the dependency to your build.gradle file.
{% include macros/mavenDep.md %}

**2:** Start coding:
~~~kotlin
import spinoza.*

fun main(args: Array<String>) {
    get("/salve") {
        "Salve Orbis Terrarum"
    }
}
~~~

**3:** Run and view:
~~~bash
http://localhost:4567/salve
~~~

## Stopping the Server
By calling the stop() method the server is stopped and all routes are cleared.

~~~kotlin
stop()
~~~

### Wait, what about starting the server?
The server is automatically started when you do something that requires the server to be started (i.e. declaring a route or setting the port).  
You can also manually start the server by calling `init()`.

## Routes
The main building block of a Spinoza application is a set of routes. A route is made up of three simple pieces:

* A **verb** (get, post, put, delete, head, trace, connect, options)
* A **path** (/hello, /users/:name)
* A **callback** (request, response) -> { }

Routes are matched in the order they are defined. The first route that matches the request is invoked.  
Always statically import Spark methods to ensure good readability:

~~~kotlin
get("/") {
    // Show something
}

post("/") {
    // Create something
}

put("/") {
    // Update something
}

delete("/") {
    // Annihilate something
}

options("/") -> {
    // Appease something
}
~~~

Route patterns can include named parameters, accessible via `params()`:
~~~kotlin
// matches "GET /saymy/foo" and "GET /saymy/bar"
// params(":name") is 'foo' or 'bar'
get("/saymy/:name") {
    params(":name")
}
~~~

Route patterns can also include splat (or wildcard) parameters. These parameters can be accessed by using the `splat()` method on the request object:

~~~kotlin
// matches "GET /say/hello/to/world"
// TODO: splat is not done
get("/say/*/to/*") {
    "Number of splat parameters: " + splat()?.size
}
~~~

### Path groups
If you have a lot of routes, it can be helpful to separate them into groups. This can be done by calling the `path()` method, which takes a `String prefix` and gives you a scope to declare routes and filters (or nested paths) in:

~~~java
// TODO: NOT DONE IN SPINOZA YET!
path("/api", () -> {
    before("/*", (q, a) -> log.info("Received api call"));
    path("/email", () -> {
        post("/add",       EmailApi.addEmail);
        put("/change",     EmailApi.changeEmail);
        delete("/remove",  EmailApi.deleteEmail);
    });
    path("/username", () -> {
        post("/add",       UserApi.addUsername);
        put("/change",     UserApi.changeUsername);
        delete("/remove",  UserApi.deleteUsername);
    });
});
~~~

## Implicitly available functionality in route context
Request and response functionality is provided implicitly in the route context:
~~~kotlin
// request related
attributes()             // the attributes list
attribute("foo")         // value of foo attribute
attribute("A", "V")      // sets value of attribute A to V
contentLength()          // length of request body
contentType()            // content type of request.body
contextPath()            // the context path, e.g. "/hello"
host()                   // the host, e.g. "example.com"
ip()                     // client IP address
params("foo")            // value of foo path parameter
params()                 // map with all parameters
pathInfo()               // the path info
port()                   // the server port
protocol()               // the protocol, e.g. HTTP/1.1
queryMap()               // the query map
queryMap("foo")          // query map for a certain parameter
queryParams()            // the query param list
queryParams("FOO")       // value of FOO query param
queryParamsValues("FOO") // all values of FOO query param
requestMethod()          // The HTTP method (GET, ..etc)
scheme()                 // "http"
servletPath()            // the servlet path, e.g. /result.jsp
session()                // session management
splat()                  // splat (*) parameters
uri()                    // the uri, e.g. "http://example.com/foo"
url()                    // the url. e.g. "http://example.com/foo"
userAgent()              // user agent

request                  // the request itself
    .headers()           // the HTTP header list
    .headers("BAR")      // value of BAR header
    .body()              // request body sent by the client
    .bodyAsBytes()       // request body as bytes
    .cookies("foo")      // the cookie `foo`
    .cookies()           // request cookies sent by the client

// response related
redirect("/example");        // browser redirect to /example
redirect("/example", 302)    // redirect to /example with status code 302
status()                     // get the response status
status(401)                  // set status code to 401
type()                       // get the response content type
type("text/xml")             // set the response content type to text/xml

response                              // the response itself
    .body()                           // get response content
    .body("Hello")                    // sets content to Hello
    cookies()                         // get map of all request cookies
    cookie("foo");                    // access request cookie by name
    cookie("foo", "bar")              // set cookie with a value
    cookie("foo", "bar", 3600)        // set cookie with a max-age
    cookie("foo", "bar", 3600, true)  // secure cookie
    removeCookie("foo")               // remove cookie
    .header("FOO", "bar")             // sets header FOO with value bar
~~~

## Sessions
Server created session functionality is implicitly available in route countext
~~~kotlin
session()                         // get session
session(create = true)            // create and return session
session().attribute("user")       // Get session attribute 'user'
session().attribute("user","foo") // Set session attribute 'user'
session().removeAttribute("user") // Remove session attribute 'user'
session().attributes()            // Get all session attributes
session().id()                    // Get session id
session().isNew()                 // Check if session is new
session().raw()                   // Return servlet object
~~~

## Halting
To immediately stop a request within a filter or route use `halt()`:
~~~kotlin
halt()                // halt 
halt(401)             // halt with status
halt("Body Message")  // halt with message
halt(401, "Go away!") // halt with status and message
~~~

`halt()` is not intended to be used inside exception-mappers. Halt throws a halt exception so if the halt is within
a try-catch catching this type of exception the halt won't work.

## Filters
Before-filters are evaluated **before each request**, and can read the request and read/modify the response.  
To stop execution, use `halt()`:
~~~kotlin
// matches all requests
before {
    if (!authenticated()) {
        halt(401, "Go Away!")
    }
}

// matches requests on /protected/*
before("/protected/*") {
    if (!authenticated()) {
        halt(401, "Go Away!")
    }
}
~~~

After-filters are evaluated **after each request**, and can read the request and read/modify the response:
~~~kotlin
after {
    response.header("foo", "bar")
}
~~~

Finally-filters are evaluated **after after-filters**. Think of it as a "finally" block.
~~~kotlin
finally { 
    response.header("foo", "bar")    
}
    
finally("/foo") {
    response.header("foo", "bar")
}
~~~

## Redirects
You can trigger a browser redirect with the redirect method on the response:
~~~kotlin
redirect("/bar")
~~~

You can also trigger a browser redirect with specific HTTP 3XX status code:
~~~kotlin
redirect("/bar", 301) // moved permanently
~~~

### Redirect API
There is also a convenience API for redirects which can be used directly without the response:

~~~kotlin
redirect {
    any(
            "/from" to "/hello",
            "/hi" to "/hello"
    )
    get(
            "/source" to "/target"
    )
    post(
            "/gone" to "/new"
    )
}
~~~

## Custom error handling {#error-handling}

### Not found (code 404) handling

~~~kotlin
notFound {
    "404 - Spinoza couldn't find what you're looking for"        
}
~~~

### Internal server error (code 500) handling
~~~kotlin
internalServerError {
    "Very internal server error"        
}
~~~

## Exception Mapping
To handle exceptions of a configured type for all routes and filters:
~~~kotlin
get("/exception") {
    throw AuthException("protection")
}

exception(AuthException::class) {
    status(401)
    response.body(exception.message)
}
~~~

## Configuration DSL
If you need to configure Spinoza use the DSL for this where you can configure port, ip, threadpool, HTTPS and static files.

~~~kotlin
// Static API
config {
    port = 5500
    ipAddress = "0.0.0.0"
    threadPool {
        maxThreads = 10
        minThreads = 5
        idleTimeoutMillis = 1000
    }
    secure {
        keystore {
            file = "/etc/secure/keystore"
            password = "hardtocrack"
        }
        truststore {
            file = "/etc/secure/truststore"
            password = "otherdifficultpassword"
        }
        needsClientCert = false
    }
    staticFiles {
        location = "/public"
        externalLocation "/var/static"
        expiryTime = 36000.seconds
        headers(
                "description" to "static content",
                "licence" to "free to use"
        )
        mimeTypes(
                "cxt" to "text/html"
        )
    }
}

// Instance API
val http = ignite {
    port = 5500
    ipAddress = "0.0.0.0"
    threadPool {
        maxThreads = 10
        minThreads = 5
        idleTimeoutMillis = 1000
    }
    secure {
        keystore {
            file = "/etc/secure/keystore"
            password = "hardtocrack"
        }
        truststore {
            file = "/etc/secure/truststore"
            password = "otherdifficultpassword"
        }
        needsClientCert = false
    }
    staticFiles {
        location = "/public"
        externalLocation "/var/static"
        expiryTime = 36000.seconds
        headers( // custom headers
                "description" to "static content",
                "licence" to "free to use"
        )
        mimeTypes(
                "cxt" to "text/html"
        )
    }
}
~~~

Initialization must be configured before route mapping. If your application has no routes, `init()` must be called manually after location is set.

## Embedded web server

Standalone Spinoza runs on an embedded [Jetty](http://eclipse.org/jetty/) web server.

### Port
By default, Spinoza runs on port 4567. If you want to set another port, use init DSL. 

### Waiting for Initialization {#awaitinit}
You can use the method `awaitInitialization()` to check if the server is ready to handle requests. This is usually done in a separate thread, for example to run a health check module after your server has started.  
The method causes the current thread to wait until the embedded Jetty server has been initialized. Initialization is triggered by defining routes and/or filters. So, if you're using just one thread don't put this before you define your routes and/or filters.

~~~kotlin
awaitInitialization() // Wait for server to be initialized
~~~

### WebSockets
WebSockets provide a protocol full-duplex communication channel over a single TCP connection, 
meaning you can send message back and forth over the same connection.

WebSockets only works with the embedded Jetty server, and must be defined before regular HTTP routes. 
To create a WebSocket route, use the webSocket DSL

~~~kotlin
webSocket("/ws") {
    opened {
        println("[opened] session = " + session)
    }
    received {
        println("[received] message = $message, session = $session")
    }
    closed {
        println("[closed] session = $session, reason = $reason")
    }
    error {
        println("[error] cause = " + cause)
    }
}
    
init() // Needed only if you don't define any HTTP routes after your WebSocket routes
~~~

## Other web server
To run Spinoza on another web server (instead of the embedded jetty server), an implementation of the interface `spark.servlet.SparkApplication` is needed. You have to initialize your routes in the `init()` method, and the following filter might have to be configured in your web.xml:

~~~xml
<filter>
    <filter-name>SparkFilter</filter-name>
    <filter-class>spark.servlet.SparkFilter</filter-class>
    <init-param>
        <param-name>applicationClass</param-name>
        <param-value>com.company.YourApplication</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>SparkFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
~~~

## GZIP
GZIP is done automatically if it's in both the request and the response headers. 
This usually only means that you have to set it in your response headers.

If you want to GZIP a single response, you can add it manually to your route:

~~~kotlin
get("/aliquid") {
    // ...
    response.header("Content-Encoding", "gzip");
}
~~~

If you want to GZIP everything, you can use an after or finally filter
~~~kotlin
finally {
    response.header("Content-Encoding", "gzip");
}
~~~

## Examples and FAQ
Examples can be found on the project's page on [GitHub](https://github.com/perwendel/spark-kotlin/blob/master/README.md#examples).
