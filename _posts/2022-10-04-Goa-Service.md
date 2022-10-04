---
layout: post
title: 'A trip to Goa: Go services the easy way'
author: stelios
tags: [programming, hands-on, api, golang]
categories: [Microservices]
featured: true
description: "In this post we are taking a close look at Goa, a contract-first microservice framework in Golang."
image: assets/images/goa-service/digjot-singh-ZemFx1ctq3o-unsplash.jpg
---

In this post we are taking a close look at Goa, a contract-first microservice framework in Golang.

# A trip to Goa and back 

Golang has surged in popularity over the last few years and for good reason.   
It has been designed from first principles for systems engineering, focusing on performance and robustness. 

Copying from [The Go Programming Language][30] book  
> Go is especially well suited for building infrastructure-like networked servers, and tools and systems for programmers, 
> but it is truly a general-purpose language(...) It has become popular as a replacement for untyped scripting languages 
> because it balances expressiveness with safety

This is a great sales pitch!  
But what about productivity?

This is where Goa comes in. 

As per its [website][31], Goa...  
> * relies on code generation to alleviate the need for reflection or repetitive coding
> * The design is the Single Source Of Truth from which both behavior and docs are derived.
> * generates code that follows best practice

To do this, Goa offers
* a domain-specific language (DSL) to express a microservice (m/s) contract  
* a code generator, for server- and client-side code. The generated code follows patterns and best practices for
  maintainability.
* first-class support for different message transports and security mechanisms, and
* plugin system for code generation extensibility.  

Before we start our hands-on tutorial, it is worth deep-diving to... 

## Goa DSL entities 

The DSL is the pillar on which Goa stands on.  
It is composed of a few basic entities which are worth explaining, as we will be using them heavily.<sup>[1](#footnote_1)</sup>
These entities are implemented as Go functions, meaning that our DSL is in fact a Go script.

![Goa concepts](../assets/images/goa-service/goa_design.png)

* `API` sits at the top, defining common global properties (name, description,...). It is composed of one or more...
* `Servers`. These are primarily referenced by auto-generated command-line clients, in lieu of full base URLs. A `Server` contains one or more...
* `Services`. This is a grouping mechanism, akin to a resource in REST. `Services` contain...
* `Methods`, which can belong to more than one `Service`. Each `Method` is composed of
  * An optional `Security` mechanism (e.g. MA-TLS, Bearer token,...)
  * An optional `Payload` and `Response` object, and
  * A mapping to one or more of the supported Transport mechanisms (`HTTP`, `GRPC`)

With DSL out of the way, let's look at...

# Our little use case

We want to create a m/s which   

* is secured by OAuth2 [Client Credentials flow][32], i.e. 
  * a token generation endpoint `/auth`. The token expires which  
  * other endpoints validate the token before executing their business logic
  * the same m/s is both Authorisation and Resource service
* multiplies 2 numbers and gives back the result (endpoint `/mul`)
* sums all numbers in a submitted document (endpoint `/sum`).<sup>[2](#footnote_2)</sup> 

# Let's get coding

![Some coding-y image](../assets/images/goa-service/joshua-reddekopp-SyYmXSDnJ54-unsplash.jpg)
> Photo by Joshua Reddekopp on Unsplash

> The code for this blog post is in repository [hello-goa][33].
> Each section below has a corresponding code branch (`v1`, `v2` etc) with the progress of the project until that point. 
> You can switch to that branch and follow along at your pace.

## v1 - Setup and service definition

We start by setting up our project and adding Goa.  
```
go mod init github.com/sgerogia/hello-goa
go get goa.design/goa/v3/
```

Our API definition is inside [branch v1 `design.go`][1]. 

We have chosen to...  
* Split our API in 2 different services: [`token`][2] (authorisation) and [`math`][3] (resource service).
* Define an explicit data type [`User`][4] for the `token` service Payload.
* Map the [`Body`][5] of the HTTP POST to the [`doc`][6] attribute of the `sum` method (uploaded JSON document for summing).
* Map the [`numbers`][7] HTTP GET path parameter to the [payload attribute][8] (path parameters to array).
* Use Goa's extraction patterns to map the [`Authorization`][9] header to the `token` attribute (for custom token validation).

## v2 - Stubs 

We can kickstart Goa's code generation with `make generate`.

The generated code ends under the `/gen` folder and is split in 3 parts:  
* [`token`][10] service
* [`math`][11] service, and
* [`http`][12] transport-related code. 

We can also go ahead and generate some example stubs with `goa example github.com/sgerogia/hello-goa/design`.<sup>[3](#footnote_3)</sup>  

The structure of the placeholder files is as follows:  
* [`/cmd/server`][13] for launching the HTTP server process, 
* [`/cmd/server-cli`][14] for the command-line client binary generation, and
* [`math.go`][15] and [`token.go`][16], for the actual logic of the 2 services. 

The placeholder files can be moved around after initial generation. They are meant to give an idea of how the 
implementation will look like. 

It is time to move to...

## v3 - Logic implementation

To better organise the code, we moved the logic placeholders to their own folder (insipidiously named [`logic`][17]). 

There is not much to say about the implementation of the 2 methods for summing and multiplying. The reader can definitely 
come up with better and cleaner approaches. 

What I will highlight are the changes we did to facilitate testing: 
* Changed the visibility (*exported*) of [`MathSrvc`][18] so it is accessible by the unit tests.
* Created a [test suite][19] to share context between tests. This will come in handy later on.

You can execute the tests with `make test`.

With logic out of the way we are ready for...

## v4 - Security implementation

The auto-generated JWT-related code gives us 2 placeholder methods, which we need to implement.  
* [`Auth`][20]: Here we validate the client credentials and generate the JWT token.
* [`JWTAuth`][21]: Called by the framework, before each of the protected endpoints.  
  This is the place to validate that the JWT token is valid and that it has the correct permissions for the current 
  endpoint (if applicable). 

What claims we include in the JWT is entirely up to the application. In our case we chose to add the standard set as 
defined by [OpenID][22].

We have also followed our previous pattern and exported the `TokenSrvc` to facilitate testing and added it to our test suite. 

The final thing to point out is the need to have a private/public keypair for the RSA signature of the JWT.  
We could have had it hard-coded in the code (e.g. see the [unit test][23]). We chose instead to demonstrate how we can 
modify the server launch code, with [additional arguments][24], custom creation of [API service objects][25] etc   

We are finally ready for an...

## End-to-end test

Let's build the binaries and run the local server.  
```bash
make build-cli
make run-local-http
```

Goa has generated a full-fledged API client out-of-the-box. Each API service (`token`, `math`) has become a separate 
CLI sub-command. You can see its usage with  
```bash
./server-cli token --help
./server-cli math --help
./server-cli math mul --help
...
```

Let's get a token with the CLI.    
> In this section we also show the equivalent cURL side-by-side.

```bash
./server-cli token auth --body '{ "password": "password", "username": "user" }'

curl -X POST http://localhost:8080/auth \
-H 'Content-Type: application/json' \
--data-binary '{ "password": "password", "username": "user" }'   
```

Let's try to multiply some numbers.   
```bash
./server-cli math mul \
  --numbers '["4", "3.543", "-2"]' \
  --token "TOKEN_FROM_THE_PREVIOUS_STEP"
  
curl http://localhost:8080/mul/4,3.543,-2 \
-H 'Authorization: Bearer TOKEN_FROM_THE_PREVIOUS_STEP'  
```

Finally, let's submit a JSON document for addition of its numeric elements.  
```bash
./server-cli math sum \
  --token "TOKEN_FROM_THE_PREVIOUS_STEP" \
  --body '{"a":{"b":4},"c":2, "d":[1, 3]}' 

curl -X POST http://localhost:8080/sum \
-H 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJ1c2VyIiwiZXhwIjoxNjY0ODgwNTY4LCJqdGkiOiJmZDQ3Zjc2MC0xN2Q3LTRlNDYtYjdjZi0yMWM1MzBjNzc0NzgiLCJpYXQiOjE2NjQ4NzY5NjgsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4MCIsInN1YiI6InVzZXIifQ.efisAM2XQdbmuNPRoYffYewH636CNgCLNh4tecnkXSGygfPvNrQhXpkeM3zA731j2VIIJqss8NeDcXPxQFbwHDdcxqmt5w0b-onNuGCFf2u7W55rWANNOCMjke5B4QSCop9waVV-eXSF70yIXPT5iNKD7SIlOv4FrNzkvNye5w4VCg7g-oZovjsZdmaLN2SfLdzyxXTBLw4TCst5SFiVzzcyhPMOevnX5mSv6p0uI_iPCO0GhdLyW20-HghazSRDI6xoj5vepuoP5_fCPdpwZUhsO75o_pl66IENhBmiovEsvpbEV5qYc0sXVKH4yk6Jcis6OCbHL3gQSVB97Sc4xTSmGkEkGWvbdyf_j4uBKRE8SVMZd93EqDJkze5Os7umKP5Nw8ws4JLuoyWab-kM1wNwTJmGNFWuhC_Tfql4blDKFZPIZOF-Yqqj67QP5f1nQ9pW1GNerbiPhTQ4noUNRtyokruyFjBhWql_0ebYf94xesvJMIa93GnVoIlSuuX9' \
--data-binary '"{\"a\":{\"b\":4},\"c\":2, \"d\":[1, 3]}"'
```

This last example allows us to see how the CLI client might "sweep under the rug" some Goa internals.  
In this case the data-type of the [`/sum` payload][26] (`String`) uses Goa's default [codec mechanism][27]. Namely, a `String` 
representation must be surrounded by double quotes. A nice improvement would be to implement a custom codec to support 
our unstructured document as JSON, without the need for quote escaping.

## Next steps

We could very well continue to expand our demo project to demonstrate Goa's productivity throughout the process.

Some new functionality ideas for the reader to implement/add 
* a new endpoint (e.g. subtraction)
* a new security mechanism (e.g. Basic Auth, MA-TLS,...)
* support for gRPC on existing endpoints
* [streaming data][29] support  
  etc

The Goa tooling allows the seamless evolution of the codebase, adding new capabilities, while minimizing waste and 
throw-away code. 

# Discussion & Parting thought

![Discussion](../assets/images/goa-service/kris-n9u9ZEoH2yM-unsplash.jpg)
> Photo by kris on Unsplash

Golang has an ever-growing share of backend systems programming.  
This is for good reason, as it occupies the sweet spot between performant execution, non-verbose syntax and just-enough 
constructs (garbage collector, interfaces,...).    

Coming from a Java background myself, I find Goa an invaluable tool to quickly become productive in the Go world.  
Goa allows the team(s) to focus on the *What* (contracts, interfaces and security), rather than over-spend time on the 
*How* (e.g. choosing [low-level frameworks][28], re-implementing wrongful one-way decisions,...). 

This allows for fast initial iterations and incremental shipping of features, while minimising one-off choices and 
throw-away code. 
This moves the needle from "Get S\*\*t Done" to "Get S\*\*t Done Right First Time". 

Not a bad thing to have!

Until next time, happy coding!

# Footnotes

1. <a name="footnote_1"></a>This section is using UML notation only for ease of expression and understanding. The actual 
Goa implementation uses different naming and relations internally. 
2. <a name="footnote_2"></a>E.g. `[1,2,3,4]` and `{"a":6,"b":4}` both have a sum of 10, `[[[2]]]` and `{"a":{"b":4},"c":-2}`
  both have a sum of 2, etc
3. <a name="footnote_3"></a>This is a one-off command. It generates files which you will be editing in the following sections. 



  [1]: https://github.com/sgerogia/hello-goa/blob/v1/design/design.go
  [2]: https://github.com/sgerogia/hello-goa/blob/v1/design/design.go#L33
  [3]: https://github.com/sgerogia/hello-goa/blob/v1/design/design.go#L60
  [4]: https://github.com/sgerogia/hello-goa/blob/v1/design/design.go#L24
  [5]: https://github.com/sgerogia/hello-goa/blob/v1/design/design.go#L102
  [6]: https://github.com/sgerogia/hello-goa/blob/v1/design/design.go#L84
  [7]: https://github.com/sgerogia/hello-goa/blob/v1/design/design.go#L136
  [8]: https://github.com/sgerogia/hello-goa/blob/v1/design/design.go#L123
  [9]: https://github.com/sgerogia/hello-goa/blob/v1/design/design.go#L99
  [10]: https://github.com/sgerogia/hello-goa/blob/v2/gen/token
  [11]: https://github.com/sgerogia/hello-goa/blob/v2/gen/math
  [12]: https://github.com/sgerogia/hello-goa/blob/v2/gen/http
  [13]: https://github.com/sgerogia/hello-goa/blob/v2/cmd/server
  [14]: https://github.com/sgerogia/hello-goa/blob/v2/cmd/server-cli
  [15]: https://github.com/sgerogia/hello-goa/blob/v2/math.go
  [16]: https://github.com/sgerogia/hello-goa/blob/v2/token.go
  [17]: https://github.com/sgerogia/hello-goa/tree/v3/logic
  [18]: https://github.com/sgerogia/hello-goa/blob/v3/logic/math.go#L18
  [19]: https://github.com/sgerogia/hello-goa/blob/v3/logic/init_test.go
  [20]: https://github.com/sgerogia/hello-goa/blob/v4/logic/token.go#L35
  [21]: https://github.com/sgerogia/hello-goa/blob/v4/logic/math.go#L34
  [22]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken
  [23]: https://github.com/sgerogia/hello-goa/blob/v4/logic/init_test.go#L15
  [24]: https://github.com/sgerogia/hello-goa/blob/v4/cmd/server/main.go#L37-L39
  [25]: https://github.com/sgerogia/hello-goa/blob/v4/cmd/server/main.go#L110-L111
  [26]: https://github.com/sgerogia/hello-goa/blob/main/design/design.go#L102
  [27]: https://goa.design/en/implement/encoding/
  [28]: https://medium.com/seek-blog/microservices-in-go-2fc1570f6800
  [29]: https://goa.design/implement/streaming/
  [30]: https://www.gopl.io/
  [31]: https://goa.design/
  [32]: https://developer.okta.com/blog/2018/06/06/node-api-oauth-client-credentials
  [33]: https://github.com/sgerogia/hello-goa/