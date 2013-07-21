---
layout: post
title: "Apache Camel with Scala: Extending DSL"
date: 2013-07-11 07:58
comments: true
categories: [Apache Camel, scala]
keywords: Apache Camel,Scala,DSL,Extending camel DSL,
description: "How to extend camel scala dsl to support custom methods." 
---


Let's take a very simple use case of integrating with a http endpoint which uses basic auth. 

``` scala

    from("someWhere")
      //Do some processing
      .setHeader("Authorization", _ => "OAuth anObfuscatedTokenString")
      .setHeader(Exchange.HTTP_METHOD, "GET")
      .to("http://someServer.com")
      .process(//Do something with response)

    from("someWhereElse")
      //Do some processing
      .setHeader("Authorization", _ => "OAuth anObfuscatedTokenString")
      .setHeader(Exchange.HTTP_METHOD, "POST")
      .to("http://someServer.com")
      .process(//Do something with response)

```

You can clearly see if have many routes like this, there will be a lot of duplication. 

<!-- more -->

So we wanted to extend the DSL so that these kind of duplications are avoided and we could come up with more readable routes.

We created an implicit class and add  method like <code>toSomeServer</code> which abstrats the details.

``` scala
object DSLImplicits {
  implicit class RichDSL(val dsl: DSL) {
    def toSomeServer = {
      dsl.setHeader("Authorization", _ => "OAuth anObfuscatedTokenString")
      	.to("http://someServer.com")
    }

    def get = dsl.setHeader(Exchange.HTTP_METHOD, _ => HttpMethods.GET.name)

    def post = dsl.setHeader(Exchange.HTTP_METHOD, _ => HttpMethods.POST.name)
  }
}

```

The above extention can be used like this.

``` scala
    import DSLImplicits.RichDSL
    //----------------------------
    from("someWhere")
      //Do some processing
      .get.toSomeServer
      .process(//Do something with response)

    from("someWhereElse")
      //Do some processing
      .post.toSomeServer
      .process(//Do something with response)
```

You can clearly see this ends with more readable route.


This is is [not possible](http://i000174.blogspot.in/2011/02/is-it-possible-to-extend-apache-camel_28.html) with java dsl. As given [here](http://stackoverflow.com/questions/8010084/camel-extend-java-dsl) extending can be possible with Groovy based DSL too.



