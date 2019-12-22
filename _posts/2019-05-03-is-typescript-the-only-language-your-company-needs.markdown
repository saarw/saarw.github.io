---
layout: post
title:  "Is TypeScript the  only language your company needs?"
date:   2019-05-03 14:30:49 +0700
categories: dev
---
(Originally posted [on Medium](https://medium.com/@saarw/is-typescript-the-only-language-your-company-needs-9f1fb11925b4))  

## Background
TypeScript is superset of JavaScript (all JavaScript programs are valid TypeScript) that adds a modern type system and compiles to regular JavaScript. Along with checking types, the compiler also helps eliminate other common errors, such as unchecked null values.   

Java has been a trusty companion for enterprise and startup work since I first used it at an internship in -96, but dedicating time to own projects where I couldn’t pass off the UI work to a front-end team led me to familiarize myself with the JavaScript landscape. While frameworks like React provide a nice experience for UI development, I remained skeptical of JavaScript code’s quality and maintainability until discovering how robust the development experience became with TypeScript.   

## The quest for a single language   
While it’s immediately appealing to be able to re-use data models and move business logic between client and server without onerous code-generation or translation steps, the justifications for efforts to use a single language for the whole stack largely come from organizational benefits to planning, training, hiring, and tooling.   

Many articles in the agile world celebrate T-shaped skills (deep specialization, the leg of the T, complemented with broad ability to help out in adjacent areas) and many companies want to hire full-stack developers. While a developer that specializes in browser behaviors is unlikely to keep up with all the latest techniques for database performance, cross-functional benefits can still be had if developers work in a single language throughout the stack.   

Earlier efforts to unify on single languages have mostly focused on writing web-UIs in server-side languages with frameworks such as Google Web Toolkit and Eclipse RAP, along with recent attempts like Elixir LiveViews. These efforts have currently failed to reach the momentum that JavaScript runtimes are seeing on the server and are also increasingly challenged by the rise web applications with offline capabilities and mobile apps, where frameworks like React Native are seeing heavy adoption.   

### So why not JavaScript?   
Experience from many large organizations is that untyped languages become unmaintainable and bug-prone at scale, with type-deficient languages such as Ruby, Clojure and Go (with generics) all prioritizing increased support for various forms of typing as the language matures. In the JavaScript world, large organizations have long been solving the problem by migrating to TypeScript and now most popular JavaScript frameworks have also added type definitions.   

While TypeScript is “only” in 12th place in language rankings such as [RedMonk’s latest ranking](https://redmonk.com/sogrady/2019/03/20/language-rankings-1-19/), it’s easy for JavaScript developers to start using the language so there is plenty of latent talent in the market and TypeScript is the 3rd “most loved” language in [StackOverflow’s recent survey](https://insights.stackoverflow.com/survey/2019#most-loved-dreaded-and-wanted) so there shouldn’t be too much resistance.   
   
## Why now?   
### Web services are going asynchronous
JavaScript has been quick to add and adopt new asynchronous async/await language constructs. Asynchronous programming models also let traditionally single-threaded runtimes, like NodeJS, use multiple processor cores more effectively for background I/O.   
   
Meanwhile Java, which has long been the dominant enterprise language unified around its servlet framework, is fragmenting its ecosystem as it moves into the asynchronous world. Java developers have to choose between numerous reactive solutions and even if they pick the winner, the code bases risk becoming obsolete unergonomic chains of lambda functions in a few years as the language finalizes support for and adopts fibers/goroutines.   
### Enterprise frameworks for TypeScript are maturing
Frameworks like [TypeORM](https://typeorm.io/) and [NestJS](https://nestjs.com/) were written with types from the start and provide constructs such as dependency-injection and database repositories that are familiar to developers with experience from Java frameworks like Spring and Hibernate.   
### The adoption of serverless and containers
As enterprises adopt serverless programming models like AWS Lambda and Google Cloud Functions, or deploy into container clusters that allow easy horizontal scaling, like Kubernetes, programming languages with rich concurrency constructs have less advantage over runtimes like NodeJS.   

## When is TypeScript a poor fit?
### Computation-heavy workloads
JavaScript runtimes like Node have poor support for concurrency and JavaScript itself isn’t the most efficient language, without primitive number types etc., so CPU-heavy use cases may still benefit from more efficient languages with richer concurrency constructs. Runtimes with longer history in the enterprise, like the Java Virtual Machine, also offer rich observability and tooling for performance tuning.   

Inclusion of WebAssembly modules, [Deno](https://deno.land/), and JVM-based TypeScript runtimes such as [Vert.x with GraalVM](https://reactiverse.io/es4x/), or serverless computation solutions may alleviate these limitations for JavaScript/TypeScript, but their story is not yet as mature.   

### Domains with poor JavaScript framework support
While there are plenty of JavaScript frameworks for writing web servers, other domains are dominated by mature and productive frameworks for other languages or benefit from custom platform-specific code.   

Data engineering frameworks have best support for JVM-based languages like Java, Scala, or possibly Python, game developers can achieve great productivity with C# and Unity, and specialized client development at resource-rich organizations may [justify native development](https://medium.com/airbnb-engineering/react-native-at-airbnb-f95aa460be1c) over productive cross-platform JavaScript frameworks like React Native.   

## Final thoughts
Disruption theory from The Innovator’s Dilemma tells us it’s natural for “toys” to mature and cover the most common uses in a market, while “serious” products move to serve increasingly specialized niche uses. Just like companies starting with databases today are unlikely to choose Oracle’s enterprise solution over alternatives considered toys a few decades ago like MySQL or Postgres, it might be worth evaluating if the language you will need to use for your front-end can now also serve the needs of your back-end.   

While TypeScript’s compatibility with JavaScript brings along a few dangerous features, like JavaScript’s equality semantics, and risks devolving into JavaScript if types aren’t used properly, these problems should be surmountable with code conventions and guidance from even moderately experienced developers.   

Node as a runtime also leaves some doubts, but it should be a no-brainer to include among options for startups or small businesses that consider Ruby or PHP. Having recently surveyed the enterprise landscape as part of helping build a web application security SaaS, it’s also clear that both Node and even TypeScript on the JVM are seeing adoption in the enterprise (even though the Java and dotNet are still the dominant players).   

For more programming language inspiration, see my earlier study on why Rust worked out well as an alternative to C/C++ for a native shared library [https://www.tcell.io/2017/06/agents-rust/](https://www.tcell.io/2017/06/agents-rust/) 
