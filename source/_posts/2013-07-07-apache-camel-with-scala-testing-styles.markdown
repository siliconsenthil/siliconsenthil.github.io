---
layout: post
title: "Apache Camel with Scala: Testing Styles"
date: 2013-07-07 15:39
comments: true
categories: [Apache Camel, scala]
keywords: Apache Camel,Scala,CamelTestSupport
description: "Testing styles for camel with scala"
---

Scala with Camel is a very powerful combination for integration. Camel has fantastic support for 
<a href="http://camel.apache.org/testing.html" target="_blank">testing</a>. It's very easy and useful to test-drive the integration. Following are few styles when it comes to testing camel with scala.

<!-- more -->

### Use CamelTestSupport, JUnit style

CamelTestSupport already has JUnit annotations to run do setup and teardown in JUnit test suite. The simplest way to get started is to use this with ScalaTest. ScalaTest already has JUnitSuite trait to have JUnit style tests. This is documented <a href="http://www.kai-waehner.de/blog/2011/06/23/apache-camel-and-scala-a-powerful-combination/" target="_blank">here</a>(see Testing section).


### Extending CamelTestSupport

If you dont like the JUnit style and wanna follow any other styles ScalaTest supports, you can create a base test support class that looks like below.

``` scala

class CustomCamelFlatTestSupport extends CamelTestSupport with BeforeAndAfterAll with FlatSpec{

  override def beforeAll() {
    setUp()
  }

  override def afterAll() {
    tearDown()
  }
}
```

We are following <i>FlatSpec</i> style of ScalaTest.

``` scala
class YourRouteSpec extends CustomCamelFlatTestSupport{
  override def createRouteBuilder = YourRouteBuilder.buildRoute

  before {
    interceptTimer()
  }

  private def interceptTimer() {
    context.getRouteDefinition("YourRouteName").adviceWith(context, new AdviceWithRouteBuilder {
      def configure() {
        replaceFromWith("direct:testEndpoint")
      }
    })
  }

  "Your Route" should "do something useful" in {
    template.sendBody("direct:testEndpoint", "Some body")
    //Assert some behaviour
    }
}
```

We override the <code>createRouteBuilder</code> with the route we wanna test. There might be needs to intercept routes for testing purpose.

But, if wanna have different routes or different interceptors in every test, this style would not suite. 

### CamelTestHelper trait

With this style you can pass differnt builders and different interceptors and the test code is a block that get the test support.

Create a trait like below

``` scala
trait CamelTestHelper{
  def withCamelTestSupport(routeBuilder: RouteBuilder, interceptor:(ModelCamelContext => Unit) = identity(_))(testWithCamelSupport: CamelTestSupport => Any){
    new CamelTestSupport{
      override def createRegistry = {
        val registry = super.createRegistry
        //Put entries to registry if needed.
        registry
      }
      override def createRouteBuilder() = routeBuilder
      setUp()
      interceptor(context)
      testWithCamelSupport(this)
      tearDown()
    }
  }
}
```

A typical way to use the trait in test would be,

``` scala
  "Your Route" should "do something useful" in {
    val directEndpoint = "direct:testEndpoint"

    val testRouteBuilder = new RouteBuilder {
      from("timer:doesNotMatter").routeId("OurRoute").to("file:/some/path"))
    }.builder

    def interceptor(context: ModelCamelContext) {
      context.getRouteDefinition("OurRoute").adviceWith(context, new AdviceWithRouteBuilder {
        def configure() {
          replaceFromWith(directEndpoint)
        }
      })
    }

    withCamelTestSupport(testRouteBuilder, interceptor){ testSupport =>
        testSupport.template.sendBody(directEndpoint, "Some content")
        //Assert files exists @ /some/path
      }
    }
  }
```

With this style we get full flexibility about testing routes.

