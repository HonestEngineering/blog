---
layout: post
title:  "Why you should not use Elixir in production"
date:   2018-11-27 17:03:00 +0200
tags: Feedback
pr_id: 12
---

[Elixir](https://elixir-lang.org/) is a pretty awesome new language based on the Erlang VM that offers some cool features like: high concurrency model, fast execution time, clustering and stateful made easy.

However, the real question is:

> Should we use Elixir in production?

<p>&nbsp;</p>

One way to prove a technology is suitable for production, is for it to verify the “Production Grade Application”.

As [explained by Kubernetes](https://kubernetes.io/blog/2018/08/03/out-of-the-clouds-onto-the-ground-how-to-make-kubernetes-production-grade-anywhere/), production grade can be summarized as:

> “anticipating accidents and preparing for recovery with minimal pain and delay”

<p>&nbsp;</p>

To simplify, we will define “Production Grade Application” into 3 guidelines:

- **Monitorable** - e.g.: *we should be able to detect and diagnose failure easily*
- **Easy to deploy** - e.g.: *deployment should be easy and automatable*
- **Stable** - e.g.: *updating our dependencies should be safe and easy*

<p>&nbsp;</p>

You are brainstorming about building your next application with Elixir?

This post will try to show you if Elixir fulfill the “Production Grade Application” guidelines.

## Stable

As young language, Elixir moves fast!

While fast evolution of a language is a good, it may lead to many collateral damages.

Elixir is not an exception to the rule.

### Compilation noises

Elixir has a cool philosophy: no breaking change for all minor releases.

However, Elixir introduces deprecations in many minor releases that leads to some compilation warnings.

Fixing compilation warning in your codebase is quick and simple, but fixing compilation warning emitted by third party libraries is not possible.

Indeed, many third party libraries want to be compatible with older Elixir versions.

This constraints add noises in the Elixir compilation logs and can mask an important warning.

However, using `mix deps.compile` before `mix compile` avoid third party libraries compilation warning, by splitting the compilation in two steps:
1. Compile dependencies (third party libraries emit compilation warning in this step)
2. Compile the application code (only application warning in this step)


### Community

The community is small and have support on tricky issue is not easy.

A simple example is the number of question posted on stackoverflow with the Elixir tag ~6 000 questions v.s. ~ 1 000 000 questions for Python languages.

The ratio of answered questions for elixir is 1/6 when  python have 1/10 unanswered questions.

In other hand Elixir have dedicated forum (named [Elixir Forum](https://elixirforum.com/)) and this forum is more active than stackoverflow.

### Divergent opinions

In the Erlang/Elixir ecosystem, some common problems are not solved in the standard library. The best example is JSON data structure manipulation.

Due to an divergent opinion on the best way to serialize and deserialize JSON data structure Erlang and Elixir does not provide a built-in module to manage JSON.

<p>&nbsp;</p>

> This resulted in a proliferation of libraries to manipulate JSON.

<p>&nbsp;</p>

Many third party libraries use different JSON library to deal with JSON format.
This can cause dependencies conflict (e.g. not compatible version) and/or in the worst case scenario a security issue.

Yet, this force Elixir developer to monitor all the sub-dependencies about JSON serialization to avoid any security issue.

This case is only one example of divergent opinions in the Elixir/Erlang community.

## Easy do deploy

Elixir, built on the top of Erlang, is using the [Supervisor pattern](http://erlang.org/doc/man/supervisor.html), described as:

> The supervisor is responsible for starting, stopping, and monitoring its child processes. The basic idea of a supervisor is that it must keep its child processes alive by restarting them when necessary.

![Supervisor Pattern]({{ "/assets/feedback-elixir-supervisor.jpg" | absolute_url }})


This allows Elixir to be deployed in two differents ways:

- Installing the Elixir runtime on the server who host the application. This deployment is equivalent as your development computer configuration.
- Using an [Erlang OTP (Open Telecom Platform)](http://erlang.org/doc/system_architecture_intro/sys_arch_intro.html) release approach that will allow you to deploy Elixir application by creating an Erlang OTP release that embed Elixir and the Elixir application to deploy.

Let’s dig into these 2 alternatives.


### Elixir runtime is on the server who host the application

Deploy by installing the Elixir runtime directly on the server is the most simple way.

Elixir runtime allow us to keep mix command and manage configuration easily, but with this method the release lost important Erlang VM features like hot upgrade, cluster mode, etc.

### By creating Erlang OTP release

Creating an Erlang OTP release can be done by hand, but this solution is [long and painful](http://erlang.org/doc/design_principles/release_structure.html).

[Distillery](https://github.com/bitwalker/distillery) is a Elixir library to simplify the creation of an Erlang OTP release.

However, the first problem is that Distillery is an external tool.

Indeed, Elixir does not provide a built-in `mix` command to create a Erlang OTP release, and this may break your deployments ([bitwalker/distillery#426](https://github.com/bitwalker/distillery/issues/426)) because the Elixir docker images don’t take care about Distillery compatibility.

The issue was introduced by a sudden upgrade of OTP version in the Elixir images.

Contrary to Golang, you should build the Erlang OTP release on the same OS system that the OS system used to run the Erlang OTP release.

This constraint adds overhead to create the Erlang OTP release (knowing what’s OS system is used to deploy the OTP release).

### Native Implemented Functions (NIF)

A painless step to deploy an Elixir application or an Erlang OTP release is the NIF third libraries.
However, you should ensure your server meets all the required dependencies.

This behaviour may introduce production application crash when the NIF libraries is updated and not the server.

[NIF](http://erlang.org/doc/tutorial/nif.html) forces to maintain an OS image that follow the Elixir code versioning to avoid any issue.

## Monitorable
Monitor Elixir/Erlang application can be hard.

Mainly because, the Elixir/Erlang application have a lot of processes that are not always linked with the main process.

Therefore, you could miss either an error or a crash.

If you come from Ruby world you can be regular with the auto-magic monitoring provided by New Relic for example. In the Elixir/Erlang world you can forget this.

Moreover, the most popular monitoring SaaS (DataDog, Librato, etc.) does not provide an official library to push metrics to their services.

Not having official support may introduce bug or performance issue in your application ([more informations](https://github.com/romul/newrelic.ex)).

Elixir/Erlang have the same issue with the open source monitoring tools like [Prometheus](https://prometheus.io/).

## Conclusion

Should you use [Elixir](https://elixir-lang.org/) in production?

To be honest, it depends on your use case and your expectations.

A lot of companies work with Elixir in production and have wonderful results, but there is also companies having bad time creating a production grade application (e.g. [Discord](https://blog.discordapp.com/scaling-elixir-f9b8e1e7c29b)).

Moreover Elixir/Erlang are languages created for a distributed system and distributed system are hard to build, maintain and monitor.

So, don’t be afraid of using Elixir to build your production project but ensure you have the right team and time to do it properly.


<style>
  h2 {
    margin-top: 3em;
  }
  h3 {
    margin-top: 2em;
  }
  table {
    width: 100%;
    border: 1px solid #cbcbcb;
    border-collapse: collapse;
    border-spacing: 0;
    margin: 1.5em 0em;
  }
  thead {
    background-color: #e0e0e0;
    color: #000;
    text-align: left;
    vertical-align: bottom;
  }
  td:first-child,  th:first-child {
    border-left-width: 0;
  }
  td, th {
    border-left: 1px solid #cbcbcb;
    border-width: 0 0 0 1px;
    font-size: inherit;
    margin: 0;
    overflow: visible;
    padding: .5em 1em;
  }
</style>


