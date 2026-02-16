# Intermingler: A Microservice Bus With Opinions

_Intermingler_ is a fancy reverse proxy and _microservice bus_. It is an interpretation of the [layered-client-server](https://roy.gbiv.com/pubs/dissertation/net_arch_styles.htm#sec_3_4_2) pattern in [Roy Fielding's REST dissertation](https://roy.gbiv.com/pubs/dissertation/top.htm). These are its goals:

* Reverse-proxy a set of microservices, and shape them into _extremely_ well-defined development targets,
* Perform essential input sanitation, output manipulation, and caching tasks,
* Reduce or eliminate `404` errors by managing the continuity and renaming history of a website's URI space.

> The name _Intermingler_ is provisional. It may end up being called something else, but it _will_ keep the behaviour.

## History

Intermingler is the result of a natural cleavage plane that has emerged in [Intertwingler](https://intertwingler.net/), which is billed as an application server with very specific characteristics. The basic unit of Intertwingler, including the engine itself, is a thing called a _handler_. These can be understood at a fundamental level as microservices that respond to one or more URIs through one or more request methods. The Intertwingler engine, then, via reverse proxy, yokes all its handlers under management together into a single contiguous address space, and manages routing, URI rewriting, and the all-important address renaming history.

The primary goal of _Intertwingler_ is to serve as a substrate for _dense hypermedia_ systems: it is intended to manage a large number of small, [reusable, machine-actionable](https://en.wikipedia.org/wiki/FAIR_data) hypermedia elements, connected to each other by a profusion of typed (and bidirectional) links. The idea is to use these like Lego to rapidly create websites with certain desirable characteristics that are hard to achieve through other paradigms. The only catch is it doesn't cover _everything_ you need to make a website.

The _rest_ of what you need to make a website, it turns out, is highly amenable to being modeled as _other_ microservices, which can be reused in other contexts. Even then though, it's necessary to merge these all together in an orderly fashion. This is what Intertwingler's core engine does, and now a simpler, higher-performing, more loosely-coupled, and thus more generically-applicable _Intermingler_ will replace it.

> [_Intertwingler_](https://github.com/doriantaylor/rb-intertwingler) will carry on its existence as a bundle of microservices that will plug into _Intermingler_.

## Development Plan

Separating the engine from its constituent microservices [was always part of the plan](https://doriantaylor.com/intelligent-heterogeneity), mainly to allow the latter to be written in whatever programming language is convenient, and interface with them over standard HTTP. The plan is to write Intermingler as a library in Rust, which lies at a sensible intersection of performance and modern memory safety. This will then touch down to the rest of the ecosystem with the following adapters:

1. First as a stand-alone daemon, for easier development,
2. then as an `nginx` module, for tighter integration into the stack,
3. then, if there are the resources, as an Apache module.

> Not only does Apache still represent about a quarter of all Web servers, its request loop was designed primarily by Fielding, whose designs inspired this entire endeavour. As such, congruence to the Apache API can be used as a measure of conceptual integrity and alignment to the principles of the REST dissertation.

The main focus in the short term, however, will be the definition of Intermingler's dynamic configuration interface, and the Handler Manifest Protocol.

## Configuration Interface

One goal for Intermingler is that it can be dynamically configured, and that the only static configuration necessary is the URL of the _dynamic_ configuration. The way this is to work is that one handler is nominated as the "lead", and provides information about how Intermingler ought to configure _itself_, as well as the other handlers. Intermingler will be configurable to poll the lead handler for updates, or otherwise define a Web hook✱ to which updates can be pushed.

> ✱ Note: it would be unwise to expose such a resource to the outside. That said, authentication is on the list to look at. Determining precisely how Intermingler is to be configured is anticipated to be a large part of the project.

Chief among the configuration for Intermingler will be the authoritative list of URIs. These are represented by a canonical identifier (which need not be HTTP(S)) and all available aliases.

## Handler Manifest Protocol

The Handler Manifest Protocol is a way for handlers (microservices) to advertise what resources (i.e., via which URIs) they control, along with:

* the request method(s),
* request and response content types,
* applicable parameters (query string, request body, etc.)
* whether they require authentication,
* additional aliases and preferred redirection targets,
* etc.

The goal of the HMP is to define a simple extension to any microservice to bring it into conformance with a handler, to facilitate not only original development, but the retrofitting of existing software.

## Copyright & License

©2026 [Dorian Taylor](https://doriantaylor.com/)

This software is provided under [the Apache License, 2.0](https://www.apache.org/licenses/LICENSE-2.0).
