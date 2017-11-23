---
layout: default
rightmenu: false
permalink: /
---

<h2 class="no-margin-top">Quick start</h2>

~~~kotlin
import spinoza.*

fun main(args: Array<String>) {
    val http: Http = ignite()

    http.get("/hello") {
        "Hello Spark Kotlin!"
    }
}
~~~

### Run and view
~~~bash
http://localhost:4567/hello
~~~

## Built for expressiveness
Spinoza is a simple and expressive Kotlin web framework DSL built for rapid development. Spinoza's philosophy (no pun intended) is to provide a truly expressive web framework with no boilerplate that results in code you could explain to your parents. Spinoza is the Kotlin sibling of Spark and is essentially Spark made for Kotlin.

There's nothing else to say really, take a look at the code and you should quickly find out if you love it or hate it. In our world the future of programming is expressive with code explaining what happens instead of how it happens. Kotlin is the perfect language for achieving this.