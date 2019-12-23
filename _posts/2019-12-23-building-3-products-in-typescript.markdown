---
layout: post
title:  "Building 3 products with backends, frontends and mobile apps in TypeScript"
date:   2019-12-23 06:30:49 +0700
categories: dev
---
Before taking a break from the enterprise big data world to build own projects I looked for tech stacks to optimize productivity for indie makers (and other non-infrastructure businesses) and wrote [Is TypeScript the only language your company needs?](/dev/2019/05/03/is-typescript-the-only-language-your-company-needs.html) back in May. This is an end-of-year review documenting technical learnings since that article from shipping 3 products and 2 open-source libraries with TypeScript.

### The projects
**Web SaaS (React frontend + NestJS backend + flushout data model)**   
PlotDash - [plotdash.com](https://plotdash.com)   
ImpactMiner - [impactminer.com](https://impactminer.com)

**iOS and Android apps (React Native, no backend)**   
ThinkCool - [thinkcool.app](https://thinkcool.app)

**Open-source libraries (available on NPM/Github)**   
[flushout](https://github.com/saarw/flushout) - Distributed data model for interactive collaboration   
[async-sequential-runner](https://github.com/saarw/async-sequential-runner) - Utility to run pausing asynchronous tasks in sequence

## Backends with NestJS and Postgres
[NestJS](https://nestjs.com/) is a web framework that provides dependency injection along with integrations and abstractions for server frameworks and middleware, such as Express, Fastify and TypeORM. Nest makes it easy to modularize the application for effective testing or separation into microservices. The programming model is familiar to users of enterprise dependency frameworks such as Java’s Guice or Spring.   

NestJS itself has full TypeScript support and working with the framework was mostly hassle-free with great documentation for both Nest and common uses of its integrations with other libraries. The quality of the documentation and TypeScript support of the libraries Nest depends on varies, but overall the libraries are mature and commonly used in production.   

### Database access with TypeORM   
NestJS provides rich integration with Object-Relational-Model library [TypeORM](https://typeorm.io/) for abstracting database access. Experience makes me use only simple functionality of ORMs that translate to easily-debuggable SQL queries, but the object layer allows swapping out or mocking databases for tests. TypeORM also has great management for migrations, to allow operations such as adding or modifying database columns in a controlled manner for production.   

TypeORM comes with support for many different databases, but I did run into some issues with missing support for surprisingly common functionality, such as getting the number of updated rows from Postgres (necessary for optimistic updates). Fortunately, TypeORM is written in TypeScript so it was fairly easy to create a fork with a workaround until the issue eventually got fixed and merged.   

#### Defensive techniques against JavaScript pitfalls
JavaScript libraries, and even some TypeScript libraries, sometimes do not guard against falsy-value handling so be careful about passing numbers or strings to such libraries as they may interpret zeroes or empty strings as missing values. For instance, the JWT authentication token integration in Nest expects you to return a user identifier or undefined/null to tell whether authentication was successful, but returning numeric user identifiers causes the authentication library to assume authentication failed if the user ID happens to be zero.

You will save yourself debugging time by ensuring that whenever you pass something through an unknown library that accepts different types, defensively wrap numbers and strings in objects to protect against falsy-evaluation (i.e. { userId: 0 }).   

JavaScript's standard number type's incompatibility with 64-bit integer types can also cause issues when integrating with systems like databases. It was easy to create a TypeORM entity field with a number type that mapped to a Postgres ‘bigint’ column, but negative numbers in the column would be interpreted incorrectly when translated to JavaScript. Be extra careful when working with 64-bit integers from other systems and check library documentation about whether the JavaScript integration uses BigInt, strings, or buffers to represent them.   

## Web frontends and mobile apps with React (Native)   
[React](https://reactjs.org/) comes integrated with support for TypeScript and worked great for single-page web applications ImpactMiner and PlotDash. [React Native](https://facebook.github.io/react-native/) allowed mobile app ThinkCool to support both iOS and Android, but the native version of React requires starting from a separate starter template to get TypeScript support. The React web and native applications render different components so the application code is not directly translatable between them (more on this below).   

Component library [Bootstrap](https://getbootstrap.com/) made styling the web frontends easy with freely available themes and responsive layout constructs to work on different screens sizes. The components rendered by React Native lack support for CSS classes and certain forms of style inheritance so there is greater variation in how different libraries style components with several component libraries offering their own theme systems. It makes sense to spend extra time before starting a React Native project to evaluate libraries for the components your application needs, and do not expect to find as many ready-made themes.   

### Hybrid web and mobile applications   
For hybrid applications that run both on the web and mobile there are compatibility libraries let you write a web application using components and styling from React Native, but not the other way around (unless you render the React web code in a mobile web view). So if you want to start a hybrid application you should start with a React Native application, but the translation libraries also rely on configuring JavaScript packagers like Babel so getting TypeScript to work may require more manual steps than starting a pure web or native project.    

### Ecosystem stability
The libraries in the React Native ecosystem were generally less mature and more fragile than for React web. I had to chase down Github issues to manually update the iOS project’s Info.plist and Android project’s build.gradle files to get the common icon library react-native-vector-icons to work on the platforms and abandoned the effort to add an Android splash screen due to doubts about the splash screen libraries (it was easy enough to add a splash screen for iOS with XCode).   
## Flushout instead of Redux and GraphQL
As single-page applications, I wanted the UIs for PlotDash and ImpactMiner to support immediate responsivity so users do not have to wait for server-side updates when they drag and drop elements or perform other interactions. The two apps were also built for remote collaboration, to support multiple users and devices updating the same model on the server.

Distributed data model flushout lets the client update a local state model for immediate responsiveness and the updates get flushed to a remote master model in the background for persistence and synchronization with updates from other clients. Using flushout for client state in the web apps meant I did not use popular state library Redux, but React’s addition of hooks and context has many developers skipping Redux in new applications and I also stuck to simple state hooks and property hierarchies in mobile app ThinkCool.

GraphQL lets clients perform detailed queries to reduce query response sizes and times, but as the models for both PlotDash and ImpactMiner are small enough to fit in browser memory and only need to load once (as single-page applications preserve state between interactions), there was little need for GraphQL's query flexibility. The web apps load the latest flushout model snapshot on initial page load and let flushout's update messages incrementally keep the client and backend models synchronized over a simple HTTP API.   

### Sequential asynchronous execution
Flushout is based on event sourcing so updates from clients need to be applied to the master model sequentially. This proved tricky in Node's highly asynchronous environment as pausing one client update to retrieve the master from the database may result in an update from another client starting to update the same master model. It was possible to sequentialize the asynchronous tasks by digging into TypeScript's asynchronous generator functions but such code quickly gets complicated, so library async-sequential-runner helps encapsulate this functionality in a simpler API.

## Final Thoughts
Most of the learnings and problems around building the products were non-technical: distribution, marketing, design, billing, Product Hunt launches, EU VAT rules etc. but TypeScript remains a winning full-stack proposition and is only getting better. The language has seen exploding adoption in the JavaScript ecosystem and most libraries I used that were not natively in TypeScript at least had type definitions. New TypeScript runtime [Deno](https://deno.land), from the creator of NodeJS, is also nearing release 1.0 and can hopefully make the language even more attractive on the server next year.
