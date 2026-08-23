# Handler Manifest Protocol

This is (currently notes for) the specification for **Handler Manifest
Protocol**, which, as you might have already guessed, is a _protocol_
for _handlers_ to advertise their _manifests_ to a conforming
microservice bus like Intermingler.

# Origin Story

Very early on in [my](https://doriantaylor.com/) career I had the
opportunity to work with [`mod_perl`](https://perl.apache.org/), which
embeds a Perl interpreter into the Apache Web server, making it
possible to write Apache modules in _Perl_, which takes much less
effort than writing them in C. Since I was young and since it was so
easy to work with, much experimentation ensued.

Server modules make it possible to manipulate the HTTP request
_itself_—and later, with Apache 2, the response—separately from
application code. And what _this_ means, in turn, is you can separate
reusable, content-agnostic operations from the application-specific
business logic, and dramatically shrink the footprints of both.

Working at the level of twiddling the HTTP exchange like this, I'm
confident, is something relatively few Web developers experience. For
starters, you pay for this capability in _coupling_, both to the
server software and to the language bindings, and those age out over
time. Second, all the major MVC frameworks recapitulate a lot of this
behaviour (and what they don't cover, the server adapters do), but
they concentrate it all in the content phase, i.e., _within_ the
respective frameworks.

> In other words, it'll be a middleware plugin for Rails or Django,
> and if not, Rack or WSGI. Since these are reusable components, most
> developers are going to get them off the shelf. However, while you
> can reuse them _within_ a given framework or programming language,
> you can't reuse them _across_ a heterogeneous system.

What is needed, then, is a way to have reusable components that
interoperate _across_ framework and language boundaries, which is to
say a protocol. I discovered at one point that the FastCGI protocol
actually has a partial provision for this, but very little seems to
implement it. Anyway, FastCGI itself appears to have fallen out of
favour. What has ostensibly happened is that the deployment surfaces
either interface directly with the code, or otherwise they are just
reverse proxies that converse over standard HTTP.

> Moreover, the toy Web servers we use to develop our applications
> have actually gotten quite good, so we usually just need something
> in front to handle the bulk of the request load.

The overarching goal of Intermingler is reliable websites that can
keep their resources online for _decades_, without succumbing to
destructive "redesigns" or switching to another platform. In other
words, a website that will never produce a `404`. To achieve this, we
need to cultivate the
[Ship-of-Thesus](https://en.wikipedia.org/wiki/Ship_of_Theseus)
property: every part of a Web application needs to be easily
replaceable without disrupting the rest of the system, including,
eventually, Intermingler itself.

Intermingler's job is to act as a _bus_ that presents a smooth,
uniform address space to the internet, while marshalling a set of
heterogeneous services on the back end. This entails a whole bunch of
configuration. It would be awfully convenient if the services
_themselves_ could advertise the resources they expose, making the bus
hot-configurable. It would furthermore be useful if it was something
you could easily retrofit into any existing Web application. The
solution I came up with is Handler Manifest Protocol.

# The Protocol

> These are note-grade.

The protocol is centered around a single structured message payload:

* what URIs the handler controls
  * canonical identifier
  * preferred path
  * alternates/naming history
  * query parameters
    * canonical identifier
    * preferred slug
    * alternates
    * value ranges
  * request methods
  * content types
    * (including in request body if applicable to method)
* transform queues
  * request queue(s?)
  * response queues
    * (plus addressable queue)
  * hooks/conditional processing
* delegates/other handlers

The configuration objects themselves are going to be addressable
resources, so any duplicate information can be referenced rather than
needing to be copied. This is going to be achieved by making the
schema canonically an RDF vocabulary, but then "downgrade" the
instance data to vanilla JSON via [JSON-LD
context](https://www.w3.org/TR/json-ld11/#the-context). In a manner
[similar to
ActivityStreams](https://github.com/w3c/activitystreams/blob/main/ns/activitystreams.jsonld)
(ie Mastodon), then, the RDF can be used to manage the _vocabulary_,
but implementations—including Intermingler itself—will only have to
understand JSON.

First choice for provisioning is to have it respond to [`OPTIONS
*`](https://datatracker.ietf.org/doc/html/rfc9110#OPTIONS) with
[`Prefer: return=representation`](https://datatracker.ietf.org/doc/html/rfc7240#section-4.2) (I
have wanted a reason to use this for a while.) It can fall back to
something like [`GET
/.well-known/…`](https://datatracker.ietf.org/doc/html/rfc8615) if the
framework in question can't hack it with `OPTIONS *`.

There also needs to be a way to tell Intermingler which handler to
consult for its initial configuration. I was originally just going to
have it as a command-line parameter but now thinking a `SRV` record
would be pretty slick. (eg Intermingler instance consults DNS for its
search domain, `_hmp._tcp.company.internal` will give host and
port. Look to
[CalDAV/CardDAV](https://datatracker.ietf.org/doc/html/rfc6764) for
precedent.)

# Definitions

* **content phase**: Where most Web development happens.
* **handler**: A _microservice_ that responds to at least one _URI_
  via at least one _request method_, and implements Handler Manifest
  Protocol.
* **intelligent heterogeneity**: When you accept that your information
  infrastructure is going to be made up of different platforms so you
  might as well be smart about it.
* **transform**: A _resource_ that responds to the `QUERY` _request
  method_, and implements a _function_ that operates over the _request
  body_ and returns it in the _response body_.
* **transform queue**: A sequence of transforms that manipulate the
  top-level request or response, fired off as subrequests.

