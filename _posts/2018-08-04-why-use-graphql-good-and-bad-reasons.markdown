---
layout: post
title:  "Why use GraphQL, good and bad reasons"
date:   2018-08-04 12:59:40 +0200
tags: Feedback
---

To be honest, GraphQL had its dose of hype the last months.

Went from “indie” to a “commercial” technology with company like [Prisma](https://www.graph.cool/), [Apollo](http://apollodata.com/optics/) or even [Amazon](https://aws.amazon.com/fr/appsync/) providing professional products and support.

As for [TypeScript](https://itnext.io/why-use-typescript-good-and-bad-reasons-ccd807b292fb) or any new technologies, comes the question:

<br>

> Should I use this technology? Do I need it?

<br>

If this question appeals to you, then this article is for you.


<p>&nbsp;</p>
<p>&nbsp;</p>



## The bad reasons

### GraphQL is a trending cool technology

Experienced developers know that we don’t use a technology because everybody uses it.

Each technology has a purpose and solves one or many particular problems.

Concerning GraphQL, the main goals of the project was to provide “A query language for your API”. Especially, allowing clients to specify what part of the data they need.

This is pretty useful when considering mobile applications which have limited bandwidth and speed.

You will see in the next "con" section that the cool GraphQL technology does not solve all issues neither for frontend or mobile side.

<p>&nbsp;</p>

### GraphQL solves all performance issues

A GraphQL API (server) implementation, out of the box, will have better performance than a standard REST API - for the clients.

Since resolvers are called in parallel, data will load faster.

However, without resolvers optimisation, like joins, batch systems, your performance gain will not be that impressive - especially against a SQL database with a small pool size.

Optimising resolvers performance can be complex since it depends on the use of your API.

Dedicated tools like [Apollo Engine](https://www.apollographql.com/engine) will help you identify bottlenecks or your API's most popular fields, but keep in mind that GraphQL will not magically improve your API performance.

<p>&nbsp;</p>

### GraphQL is the new REST

GraphQL and REST are both very different things, GraphQL is a language and a technology, REST is an architecture pattern.

Since they both introduced two different solutions, they can be complementary, and in short:

<br>

> GraphQL is not the end of REST.

<br>

Although GraphQL is very useful to solve complex data exchange, it is “over-engineered” to use it for simple or standard use cases.

For example, microservices or non-user facing APIs, like APIs exposing data for Business Intelligence or other purposes has no need to provide heavy GraphQL API - after all REST is just HTTP with convention, so, simple to implement and use.

We will see in the next "con" that GraphQL is not so present in ecosystems other that JavaScript.
Basically, GraphQL solves everything.

First, as said in the previous section, GraphQL requires sending JSON to the API in order to get data.

This can look dumb but, using GraphQL on the client side often requires to use a dedicated library to tackle the language complexity - that’s why we often talk about Apollo GraphQL rather than just GraphQL.

That’s also why, compared to the simplicity of REST, you need to be sure that your problematics are linked to what GraphQL can solve (see “good reasons”).

<p>&nbsp;</p>

Secondly, as said in the second point, a GraphQL API server needs optimisations and maintenance.

Making the graph-oriented way of getting data fit over an SQL database can be tricky.

And, since it enables easier use on client side, GraphQL will directly move that responsibility to the server side.

Now, the server has to deal with the client data queries and handle the complexity.

<p>&nbsp;</p>

Finally, GraphQL is born from the JavaScript ecosystem.

You will feel a real Developer Experience loss by using GraphQL outside a JavaScript ecosystem.
Let’s take the example of Ruby.

Even if many people are deeply involved in its Ruby implementation by maintaining graphql-ruby or graphql-client, these involve using Threads and very declarative syntax (class/module based) - are you familiar with Thread safe programming in Ruby?

Even in others languages or ecosystems, the edge of GraphQL resides in JavaScript and especially in the Apollo project.

<p>&nbsp;</p>
<p>&nbsp;</p>

## The good reasons

### Mobile first User Experience

Definitely, GraphQL is a game changer for building rich UI/UX based mobile applications.

Forget the “all-in-one” or preload REST endpoint and enjoy the force of GraphQL: loading what you need, when you need it, in a glance.

![GraphQL compared to REST for mobile]({{ "/assets/REST-vs-GraphQL-example.png" | absolute_url }})

<p>&nbsp;</p>

### GraphQL will help you handle complex schemas

#### For User Interfaces

Is your application based on a Schema using a lot of nested models and associations?

Does your designer deliver a complex UI showing many resources all-in-one place?

Since modern UI applications become more and more complex with rich user experience using data far away from the REST CRUD based API, GraphQL comes with a nice solution to Domain Driven APIs.

By Domain Driven API, building APIs that exposes data schema dedicated to its usage (like computed values, flattened relations)

Just take the following chat model example:

![Example chat models]({{ "/assets/chat-example-models.png" | absolute_url }})

Basically a chat can be:
- one-on-one conversation
- group conversation
- conversation around a post (images, videos or notes)

<p>&nbsp;</p>

Let’s now consider the following associated UI mockup:

![Example chat UI mockup]({{ "/assets/chat-component-mockup.png" | absolute_url }})

You can now easily understand that developing and maintaining these kind of components, especially with realtime and paging can be hard with REST APIs since you will have to deal with many associations, up to 3 nesting levels.

![Example chat UI mockup with models]({{ "/assets/chat-models-with-chat-component.png" | absolute_url }})

#### On the server-to-server side

This example is also true on the “server to server” side.

GraphQL is not a front-end client only, it can also be used on a server-to-server scenario.

See the [Github Awards](http://git-awards.com/) website that ranks Github users by counting stars on their repos.

This website has to fetch all users' repos and count stars, which is totally feasible with REST.

However, imagine that the maintainers want to update the ranking formula by using:

- user followers count
- number of forks for all repos
- include repos from organisations that users belongs to.

Then, it will become hard to do with REST.

Hopefully, Github is one of the rare company that provides a [public GraphQL endpoint](https://developer.github.com/v4/).

Fetching all this data will be easy to do and maintain - without even talking about performance.

<p>&nbsp;</p>

This is one of the many “data synchronization” process that is easier to do with GraphQL.

<p>&nbsp;</p>

### GraphQL as a microservice orchestration solution

GraphQL, especially when used with Apollo Server, provides two killer features for orchestration.

First, abstracting many REST APIs behind a unified - domain driven - GraphQL Schema.

Apollo Server [provides REST DataSource](https://www.apollographql.com/docs/apollo-server/features/data-sources.html#REST-Data-Source) in order to implement REST API orchestration.

You will, in a glance, be able to provide to a user-facing API, a unified and optimised public API of your microservices.

This will allow you to have a better handling of versioning, maturity levels of your APIs through a decoupled architecture.

Finally, take a look at schema stitching.

Schema stitching is the process of creating a single GraphQL schema from multiple underlying GraphQL APIs.

This feature, [only available with Apollo Server](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html), is similar to REST API abstractions.
However, the cool thing is that, by unifying many GraphQL APIs into one, Apollo will forward types for you, no need to reimplement all abstracted sources resolvers. 🚀

When it comes to microservices orchestration, GraphQL provides you with two different and complete ways of providing the best architecture for your back-end.

<p>&nbsp;</p>

### GraphQL will give your team a better Developer Experience

If you need to remember one good reason to use GraphQL, stick to Developer Experience.

GraphQL enhance your developers Experience in many ways, from the language to the ecosystem.

#### Descriptive Language to handle complex data
You can feel that GraphQL is inspired from the JSON syntax, it makes this language easy to learn and understand and also easy to organise in the code (for example using dedicated files)

Since your data requests are now expressed by a language, stored via strings, it becomes instantly understandable which data your code is manipulating.

#### Loading state management simplified

Despite the fact that Apollo can be seen as the “Achilles' heel” of GraphQL, this library bring exactly what you expect from a framework: more time to focus on the domain centric features, on the actual product.

Apollo React will handle for you two main technical difficulties.

Client-side caching by maintaining for you a cache of requested data and keep it updated using Observables.

The nifty thing is the ability for Apollo to update related objects across many queries.

The second problem-solved is the Query states mechanism provided by the `<Query>` component.

Apollo enables developers to specify a caching strategy (specific at each query component) and also provides the ability to view components with usefuls informations about the data (loading, state, error).

This customisation and information allow to build fine-tuned components with great UX.

At last word about Apollo will mention the ability to customise the default behaviour of Apollo by providing custom links, or even a custom cache implementation.

Definitely worth taking a look!

<p>&nbsp;</p>

#### Your are manipulating types
Finally, remember that REST exposes JSON data while GraphQL exposes types.
This subtlety creates all the difference when it comes to parse and sent data to an API.

Since everything is typed in GraphQL, it allows Apollo React to validate data sent on the client side, before reaching the API.

Finally, Apollo provides type generation tools for TypeScript, Flow and also Swift for mobile development.
This is an awesome feature because, if your clients use types provided by your API, then your clients business logic become safer (for example: compared to plain JS)

<p>&nbsp;</p>
<p>&nbsp;</p>

## Conclusion

### Pros

**Helps providing a mobile first user experience**

Since mobile apps have rich UIs and are used over slow networks, GraphQL will help you to load only the relevant data without killing the Developer Experience.


**Helps you handle complex schema**


Since modern UI applications become more and more complex with rich UX using data far away from the REST CRUD based API, GraphQL allows you to combine data from different sources.


**Microservices orchestration solution**

GraphQL, especially with Apollo Server, provides many features to hide backend complexity from clients.


**Give your team a better Developer Experience**

GraphQL is not just a new way to query data, it also enhances the way your team (front-mobile/back) will work together.

GraphQL also facilitates the creation of great UX by making loading and dealing with data easier.

<p>&nbsp;</p>

### Cons

**Using a technology based on its popularity is not enough**

Since a technology solves one or many specific issues, popularity is never sufficient to validate a technology choice.


**GraphQL will not solve all your performance issues out the box**

GraphQL isn’t a performance optimiser tool, you are still responsible to improve performance.


**GraphQL is not REST and will not replace it**

GraphQL and REST are two different things and deserve their place in the web technologies world.

**GraphQL won’t solve all your problems**

GraphQL will help you build rich mobile or web clients.

It can also helps you to improve data related backend - like indexing jobs.


<p>&nbsp;</p>
<p>&nbsp;</p>

<div style="text-align: right;">
<em>
Honest Engineers.
</em>
<div>
