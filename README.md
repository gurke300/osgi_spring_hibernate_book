# Moving legacy code based on Hibernate 4 and Spring 4 to an OSGi friendly version
[OSGi](www.osgi.org). Since I heard of OSGi I was convinced of the concept providing a dynamic runtime environment for software modules. In 2009 a former colleague of me and I were thinking about a plugin mechanism to enhance the functionality of our core component without tightly couple the code base to these (optional) functional extensions.
But at this time it was not so easy to find good material about OSGi and so we decided to use  the [**J**ava **P**lugin **F**ramework](http://jpf.sourceforge.net/) which felt more familiar to us.

As I heard of the OSGi Enterprise specification approach and the Spring team entering the OSGi world with its Spring DM server I really thought that OSGi would soon become a standard in the Java enterprise universe. But then Springsource decided to quit their efforts and submitted their Spring DM Server Solution to the eclipse community where it is going on as Eclipse Virgo. Besides that there are quite a few voices (from known experts but also keep an eye on the dates) that will not encourage you to use OSGi technology.
> <span style="color:red"> **TODO: Add some links here**</span>

But you could also find different voices saying the opposit.
> <span style="color:red"> **TODO: Add some links here**</span>**

To make one thing clear: I write this book without judging about OSGi. In fact I am still convinced about this technology. With this book I just would like to show which problems I faced and how I could solve them. But as a newbie I find it hard to gather good entry material to this technology and the development. And of course you run into troubles that are not mentioned in the books or for which it is hard to find approaches to fix them.
In order to understand the technology I highly recommend these books that I used to understand OSGi. A very good book from O'Reilly is:
* [Building Modular Cloud Apps with OSGi](http://shop.oreilly.com/product/0636920028086.do)

but also Manning offers very useful Books:
* [OSGi in Depth](http://www.manning.com/alves/) and
* [Enterprise OSGi in Action](http://www.manning.com/cummins/).

Each of the books covers somehow running Java EE aspects. But each of them is focusing on different parts. For a Java developer having quite good knowledge of [Spring framework](http://projects.spring.io/spring-framework/) **Enterprise OSGi in Action** could be very good starting point. The authors are using Blueprint for defining application logic which looks very familiar to developers using Spring. But if it comes to configuring your application using the OSGis ConfigurationAdmin or if you would like to use OSGi's own event infrastructure **OSGi in Depth** will be good choice. If you are creating/using ReST Services **Building Modular Cloud Apps with OSGi** is very enlightened.

But: in my case I was running into some difficulties caused by some restrictions to our platform which I will lay out throughout the chapters.

At the end of chapter 5 we will end up with a solution containing:
* modularized Maven project using Spring 4 and Hibernate 4 dependencies
* Spring bean definition replacement via OSGi Blueprint bean definitions (apache aries)
 * plain hibernate 4 integration based on blueprint configured spring implementation beans
* jta transaction configuration
* integration tests using [Pax Exam 4](https://ops4j1.jira.com/wiki/display/PAXEXAM4/Pax+Exam) and [Apache Felix](http://felix.apache.org/)

## Intention of this book
I write this book because I would like to share my experiences in adopting to OSGi and to help others to avoid certain pitfalls.

When you are a professional developer it happens not that often that you can start on the green field and build an application from scratch. More common is that you have to maintain code. Code that has been developed by others, whom in the worst case are not working at your organization anymore. You need to deal with historically grown code which uses libraries that may have been developed several years ago. By **migrating** such a code base **to an "OSGi-friendly" form**, I mean an application design that is capable of running within an OSGi container. This has not only impact from an architectural point of view, but how you deal with dependencies or load classes, too. This can become quite tricky. In fact a lot of Java libraries that are widely used are not built for an OSGi runtime environment. And that is may be the biggest problem of OSGi. Not OSGi itself but the 3rd party libraries are making life in the OSGi world difficult.

Me too were taking over a code base of an ECommerce platform, that is tightly coupled to Spring and Hibernate. I was doing the migration of the platform from the major version 3 to 4 for both frameworks (whereas the Hibernate one required more work to do)). Moving on to OSGi as the runtime container was much more difficult, allthough I didn't find OSGi complicated itself but to fix the errors I was running in caused by the changed runtime environment.
And as mentioned in the book's title we will focus on the persistence part of the application.

Additionally I would be very happy if you could give me feedback and or participate on this book. I would like to help others in using OSGi and to collect solutions/patterns for web development based on OSGi (with your participation very very appreciated).

## Structure of the book
**Chapter 1** will describe the legacy project and of which parts it is build up. It will contain a domain model that consists of quite a few entities. I will point out some facts that may challenge us on our refactoring.

**Chapter 2** will list the key concepts about OSGi from a high level perspective. This is important because I found it hard to get easy to understand documentation. And I am going to talk about IDE Support.

**Chapter 3** prepares a small template project using [Pax Exam 4](https://ops4j1.jira.com/wiki/display/PAXEXAM4/Pax+Exam) to enable OSGi integration tests. In this chapter I will share some thoughts about using the *Maven Repository* vs. *OSGi Bundle Repository*.

**Chapter 4** introduces Blueprint for replacement of the Spring configuration files with Blueprint's one.

**Chapter 5** will cover Hibernate's configuration relying on Spring implementations but using Blueprint.
