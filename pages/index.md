---
layout: default
rightmenu: false
permalink: /
---

<h2 class="no-margin-top">Quick start</h2>
_put this in main function:_
~~~kotlin
get("/salve") {
    "Salve Orbis Terrarum"
}

~~~

### Run and view
~~~bash
http://localhost:4567/salve
~~~

## Built for expressiveness
Spinoza is a web DSL in Kotlin built for fully expressiveness. 
Spinoza's philosophy (pun intended?) is to provide a truly expressive web framework with no boilerplate that results in code you could explain to your parents.
Spinoza is the sibling of Spark and is essentially Spark made for Kotlin.

There's nothing else to say really, take a look at the code and you should quickly find out if you love it or hate it. 
In our collective mind the future of programming is expressive (declarative) with code explaining _what_ happens instead of _how_ it happens. 
Kotlin is the perfect language for achieving this and Spinoza makes use of this to 100%. 