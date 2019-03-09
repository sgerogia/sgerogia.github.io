---
layout: post
title: Creating a SpringBoot and Kotlin micro-service
excerpt_separator: <!--more-->
tags: [SpringBoot, Kotlin, Micro-services]
---

## Introductions

![Spring boot](../images/flight-stats/meg-kannan-245874-unsplash.jpg)
> Photo by Meg Kannan on Unsplash

I like [Spring][1].  
I like [Kotlin][2]. 
I like how the 2 [can work together][6].

This post could have ended here and I would be happy.

However there would not be much value, other than demonstrating a 
bad [spartan][3] approach to posting. 

Perhaps showing a simple showcase of how to get up and running with both
would be better :-) 

First a quick intro:

[SpringBoot][4] (SB) is a framework on top of the Spring framework.  
What this lame sentence<sup>haven't read it, all lameness is mine</sup> means 
is that SB is not something new in terms of dependency injection and the like.  
Instead, it takes the [convention over configuration][5] approach, setting 
all of Spring's switches and levers to sensible defaults. 

In their own, more eloquent words

> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".
>  
>  We take an opinionated view of the Spring platform and third-party libraries so you can get started with minimum fuss. Most Spring Boot applications need very little Spring configuration.
>  
>  **Features**
>  * Create stand-alone Spring applications
>  * Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)
>  * Provide opinionated 'starter' dependencies to simplify your build configuration
>  * Automatically configure Spring and 3rd party libraries whenever possible
>  * Provide production-ready features such as metrics, health checks and externalized configuration
>  
>  Absolutely no code generation and no requirement for XML configuration 

Kotlin has been around for a number of years now.  
A product of [JetBrains][7], it builds upon a number of lessons learnt 
over years of Java experience. What this means is that it has been created
as a natural evolution of Java, both syntactically as well as at the 
runtime library level. This makes it a breeze to work with when coming 
from a Java development background.  
Since it compiles to JVM bytecode, it can utilize existing Java libraries.  
Which is exactly our case here.

The one issue I encountered while working with Kotlin and SB, was settling
down on a configuration which works.

SB evolves fast.  
Kotlin evolves fast.  
JetBrains and Pivotal are both thankfully much faster than the [JCP][8] 
resulting in [compounding effects][9]. This also results in online articles and 
how-to guides becoming stale much faster. 

Enter [Spring Initializr][10]. 
<!--more-->

## Setting things up

![Code](../images/flight-stats/arget-1140288-unsplash.jpg)
> Photo by Arget on Unsplash

Initializr is a god-send. 

Are you starting a new multi-month, enterprise project? Initializr.  
Are you doing a pair-programming interview and you are under extreme 
time-pressure? Initializr.  

It is [available online][11], ready to be used.

![Initializr UI](../images/flight-stats/initializr.jpg)

A couple of values typed and you can download the zipped project.

Even better, you can hit a REST endpoint and download from the command line.

```
curl https://start.spring.io/starter.tgz \
  -d groupId=com.affinitycrypto.dizzy \
  -d artifactId=flight-status-service \
  -d version=0.0.1-SNAPSHOT \
  -d name=FightStatus \
  -d description='Dizzy System - Flight Status service' \
  -d packageName=com.affinitycrypto.dizzy.flightstatus \
  -d dependencies=web,actuator,h2,data-jpa \
  -d language=kotlin \
  -d type=maven-project \
  -d baseDir=flight-status-service \
  | tar -xzvf -
```

Very handy indeed!

## Let's add some stuff

I want to create a simple m/s
which implements a small part of the 

The command-line above created a project



   [1]: https://spring.io/projects
   [2]: https://kotlinlang.org/
   [3]: https://dictionary.cambridge.org/dictionary/english/spartan
   [4]: https://spring.io/projects/spring-boot
   [5]: https://en.wikipedia.org/wiki/Convention_over_configuration
   [6]: https://kotlinlang.org/docs/tutorials/spring-boot-restful.html
   [7]: https://www.jetbrains.com/
   [8]: https://en.wikipedia.org/wiki/Java_Community_Process
   [9]: https://www.investopedia.com/walkthrough/corporate-finance/3/discounted-cash-flow/compounding.aspx
   [10]: https://github.com/spring-io/initializr/
   [11]: https://github.com/spring-io/initializr/
   [12]: 