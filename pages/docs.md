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
* [Request](#request)
* [Response](#response)
* [Query Maps](#query-maps)
* [Cookies](#cookies)
* [Sessions](#sessions)
* [Halting](#halting)
* [Filters](#filters)
* [Redirects](#redirects)
* [Error Handling](#error-handling)
* [Exception Mapping](#exception-mapping)
* [Static Files](#static-files)
* [ResponseTransformer](#response-transformer)
* [Views and Templates](#views-and-templates)
* [Embedded Webserver](#embedded-web-server)
  * [- Port](#port)
  * [- Secure](#secure)
  * [- ThreadPool](#threadpool)
  * [- Await initialization](#awaitinit)
  * [- WebSockets](#websockets)
* [Other Webserver](#other-web-server)
* [GZIP](#gzip)
* [Javadoc](#javadoc)
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

## Static Files
You can assign a folder in the classpath serving static files with the `staticFiles.location()` method. Note that the public directory name is not included in the URL.  
A file `/public/css/style.css` is made available as `http://{host}:{port}/css/style.css`

~~~java
// root is 'src/main/resources', so put files in 'src/main/resources/public'
staticFiles.location("/public"); // Static files
~~~

You can also assign an external folder (a folder not in the classpath) to serve static files by using the `staticFiles.externalLocation()` method.\\

~~~java
staticFiles.externalLocation(System.getProperty("java.io.tmpdir"));
~~~

Static files location must be configured before route mapping. If your application has no routes, `init()` must be called manually after location is set.

### Cache/Expire time
You can specify the expire time (in seconds). By default there is no caching.

~~~java
staticFiles.expireTime(600); // ten minutes
~~~

### Setting custom headers
~~~java
staticFiles.header("Key-1", "Value-1");
staticFiles.header("Key-1", "New-Value-1"); // Using the same key will overwrite value
staticFiles.header("Key-2", "Value-2");
staticFiles.header("Key-3", "Value-3");
~~~

## ResponseTransformer {#response-transformer}
Mapped routes that transform the output from the handle method. This is done by extending the `ResponseTransformer` object and passing it to the mapping method. Example of a route transforming output to JSON using Gson:

~~~java
import com.google.gson.Gson;

public class JsonTransformer implements ResponseTransformer {

    private Gson gson = new Gson();

    @Override
    public String render(Object model) {
        return gson.toJson(model);
    }

}
~~~

and how it is used (MyMessage is a bean with one member 'message'):

~~~java
get("/hello", "application/json", (request, response) -> {
    return new MyMessage("Hello World");
}, new JsonTransformer());
~~~

You can also use Java 8 method references, since ResponseTransformer is an interface with one method:

~~~java
Gson gson = new Gson();
get("/hello", (request, response) -> new MyMessage("Hello World"), gson::toJson);
~~~

## Views and Templates
Spark has community-provided wrappers for a lot of popular template engines:

<div class="template-engine-list" markdown="1">
* [Velocity](#velocity) (very mature, feature rich, great IDE support)
* [Freemarker](#freemarker) (very mature, feature rich, great IDE support)
* [Mustache](#mustache) (mature, decent IDE support)
* [Handlebars](#handlebars) (mature, decent IDE support)
* [Jade](#jade) (mature, decent IDE support)
* [Thymeleaf](#thymeleaf) (mature, feature rich, decent IDE support)
* [Pebble](#pebble) (we know very little about this)
* [Water](#water) (we know very little about this)
* [jTwig](#jtwig) (we know very little about this)
* [Jinjava](#jinjava) (we know very little about this)
* [Jetbrick](#jetbrick) (we know very little about this)
</div>

There are two main ways of rendering a template in Spark. You can either call render directly in a standard route declaration (recommended), or you can provide the template-engine as a third-route parameter (likely to be removed in the future):

~~~java
// do this
get("template-example", (req, res) -> {
    Map<String, Object> model = new HashMap<>();
    return new VelocityTemplateEngine().render(
        new ModelAndView(model, "path-to-template")
    );
});
~~~

~~~java
// don't do this
get("template-example", (req, res) -> {
    Map<String, Object> model = new HashMap<>();
    return new ModelAndView(model, "path-to-template");
}, new VelocityTemplateEngine());
~~~

It can be helpful to create a static utility method for rendering:
~~~java
get("template-example", (req, res) -> {
    Map<String, Object> model = new HashMap<>();
    return render(model, "path-to-template");
});

// declare this in a util-class
public static String render(Map<String, Object> model, String templatePath) {
    return new VelocityTemplateEngine().render(new ModelAndView(model, templatePath));
}
~~~
{% assign templateEngines = "velocity,freemarker,mustache,handlebars,jade,thymeleaf,pebble,water,jtwig,jinjava,jetbrick" | split: "," %}

{% for templateEngine in templateEngines %}
<div class="template-engine" markdown="1">
### {{templateEngine | capitalize}} {#{{templateEngine}}}
Renders HTML using the {{templateEngine | capitalize}} template engine. 
Source and example on [GitHub](https://github.com/perwendel/spark-template-engines/tree/master/spark-template-{{templateEngine}}).

<div class="language-xml highlighter-rouge" markdown="1">
~~~markup
<dependency>
    <groupId>com.sparkjava</groupId>
    <artifactId>spark-template-{{templateEngine}}</artifactId>
    <version>{{site.templateversion}}</version>
</dependency>
~~~
</div>
</div>
{% endfor %}

## Embedded web server

Standalone Spark runs on an embedded [Jetty](http://eclipse.org/jetty/) web server.

### Port
By default, Spark runs on port 4567. If you want to set another port, use `port()`. 
This has to be done before declaring routes and filters:

~~~java
port(8080); // Spark will run on port 8080
~~~

### Secure (HTTPS/SSL) {#secure}

You can set the connection to be secure via the `secure()` method.\\
This has to be done before any route mapping:

~~~java
secure(keystoreFilePath, keystorePassword, truststoreFilePath, truststorePassword);
~~~

If you need more help, check out the [FAQ](#enable-ssl).

### ThreadPool

You can set the maximum number of threads easily:

~~~java
int maxThreads = 8;
threadPool(maxThreads);
~~~

You can also configure the minimum numbers of threads, and the idle timeout:

~~~java
int maxThreads = 8;
int minThreads = 2;
int timeOutMillis = 30000;
threadPool(maxThreads, minThreads, timeOutMillis);
~~~

### Waiting for Initialization {#awaitinit}
You can use the method `awaitInitialization()` to check if the server is ready to handle requests. This is usually done in a separate thread, for example to run a health check module after your server has started.  
The method causes the current thread to wait until the embedded Jetty server has been initialized. Initialization is triggered by defining routes and/or filters. So, if you're using just one thread don't put this before you define your routes and/or filters.

~~~java
awaitInitialization(); // Wait for server to be initialized
~~~

### WebSockets
WebSockets provide a protocol full-duplex communication channel over a single TCP connection, meaning you can send message back and forth over the same connection.

WebSockets only works with the embedded Jetty server, and must be defined before regular HTTP routes. To create a WebSocket route, you need to provide a path and a handler class:

~~~java
webSocket("/echo", EchoWebSocket.class);
init(); // Needed if you don't define any HTTP routes after your WebSocket routes
~~~

~~~java
import org.eclipse.jetty.websocket.api.*;
import org.eclipse.jetty.websocket.api.annotations.*;
import java.io.*;
import java.util.*;
import java.util.concurrent.*;

@WebSocket
public class EchoWebSocket {

    // Store sessions if you want to, for example, broadcast a message to all users
    private static final Queue<Session> sessions = new ConcurrentLinkedQueue<>();

    @OnWebSocketConnect
    public void connected(Session session) {
        sessions.add(session);
    }

    @OnWebSocketClose
    public void closed(Session session, int statusCode, String reason) {
        sessions.remove(session);
    }

    @OnWebSocketMessage
    public void message(Session session, String message) throws IOException {
        System.out.println("Got: " + message);   // Print message
        session.getRemote().sendString(message); // and send it back
    }

}
~~~

## Other web server
To run Spark on another web server (instead of the embedded jetty server), an implementation of the interface `spark.servlet.SparkApplication` is needed. You have to initialize your routes in the `init()` method, and the following filter might have to be configured in your web.xml:

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
GZIP is done automatically if it's in both the request and the response headers. This usually only means that you have to set it in your response headers.

If you want to GZIP a single response, you can add it manually to your route:

~~~java
get("/some-path", (request, response) -> {
    // code for your get
    response.header("Content-Encoding", "gzip");
});
~~~

If you want to GZIP everything, you can use an after-filter
~~~java
after((request, response) -> {
    response.header("Content-Encoding", "gzip");
});
~~~

## Javadoc
### javadoc.io
Javadoc is available at [javadoc.io](http://javadoc.io/doc/com.sparkjava/spark-core).

### Build it yourself
After getting the source from [GitHub](https://github.com/perwendel/spark) run:

~~~bash
mvn javadoc:javadoc
~~~
The result is put in /target/site/apidocs


## Examples and FAQ
Examples can be found on the project's page on [GitHub](https://github.com/perwendel/spark/blob/master/README.md#examples).

### How do I upload something?
_Note: This applies to the standard configuration of Spark (embedded jetty). If you're using Spark with some other webserver, this might not apply to you._

To upload a file you need a form and a post handler. First, create a form with the correct enctype, and an input field with the type "file" and a name of your choice (here "upoaded_file"):
~~~html
<form method='post' enctype='multipart/form-data'>
    <input type='file' name='uploaded_file'>
    <button>Upload picture</button>"
</form>"
~~~

For Spark to be able to extract the uploaded file, you have to set a specific request attribute, which allows to use the `getPart()` method on the raw request:
~~~java
post("/yourUploadPath", (request, response) -> {
    request.attribute("org.eclipse.jetty.multipartConfig", new MultipartConfigElement("/temp"));
    try (InputStream is = request.raw().getPart("uploaded_file").getInputStream()) {
        // Use the input stream to create a file
    }
    return "File uploaded";
});
~~~

The Java-IO-stuff is left out as it's not Spark-specific, but you can see a fully working example [here](https://github.com/tipsy/spark-file-upload).

### How do I enable SSL/HTTPS?
Enabling HTTPS/SSL requires you to have a keystore file, which you can generate using the Java keytool [(→ oracle docs)](https://docs.oracle.com/cd/E19509-01/820-3503/ggfen/index.html). Once you have the keystore file, just point to its location and include its password.

~~~java
String keyStoreLocation = "deploy/keystore.jks";
String keyStorePassword = "password";
secure(keyStoreLocation, keyStorePassword, null, null);
~~~
Check out the [fully working example](https://github.com/tipsy/spark-ssl) on GitHub if you need more guidance.

### How do I enable logging?
You might have seen this message when starting Spark:
~~~bash
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
~~~

To enable logging, just add the following dependency to your project:
~~~xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.21</version>
</dependency>
~~~

### How do I enable automatic refresh of static files?
If you use `staticFiles.location()`, meaning you keep your static files in the classpath, static resources are copied to a target folder when you build your application. This means you have to make/build your project in order to refresh static files. A workaround for this is to tell Spark to read static files from the absolute path to the src-directory. If you do this you will see changes instantly when you refresh, but if you build a jar file it will only work on your computer (because of the absolute path). So, **only use this during development.**

~~~java
if (localhost) {
    String projectDir = System.getProperty("user.dir");
    String staticDir = "/src/main/resources/public";
    staticFiles.externalLocation(projectDir + staticDir);
} else {
    staticFiles.location("/public");
}
~~~
