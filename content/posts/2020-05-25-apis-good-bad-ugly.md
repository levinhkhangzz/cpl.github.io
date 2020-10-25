---
title: "APIs, the good, the bad and the ugly"
description: "Good APIs, Bad APIs, Horror APIs, and a personal rant + opinionated designs on APIs"
tags: [technical, guide, api, http, system-design, software, rant]
date: 2020-05-25
---

## Unrelated rambling

Long time no blog, 2020 is the year that keeps on giving. While some people consider this to be a pretty chaotic and horrible year, I've been making the best of it.

* Learning to bake bread and other such things
* Learning frontend (React)
* Dipping my toes into GraphQL
* Quiting my corporate job

The usual.

![2020](https://media.giphy.com/media/1rNWZu4QQqCUaq434T/giphy.gif)

## Prologue

In my past year working in the corproate world, I had to integrate with both internal and external APIs on a daily basis. Whilst doing so, I've encountered API ranging from "We have a library for every language in existance" to "ƒuçk you, we don't speak HTTP here, use XML over TCP" and everything inbetween:

* Single GET endpoint and everything is passed as query args
* APIs who respond with TEXT or HTML (have fun parsing)
* Non-deterministic state
* Custom status codes that span over an entire `uint64`
* Making 5 API calls to get one thing done
* What's consitency? Here have the same field in 3 types: `3` `3.14` `"3.1"`
* 5XX, server is busy doing body shots or something, who knows

As such, I had the honor of using some really smart APIs that inspired me to design better interfaces myself, and then there are the cråppy APIs (some of them are critical to the Internet as we know it) which gives me nightmares and makes me think that [2038](https://en.wikipedia.org/wiki/Year_2038_problem) will be the *end*.

In this post, I'll cover some of my frustrations/rants with APIs, share with you the horrors I have encountered and give my two cents on the matter of API good practices and design.

## What makes an API useful?

Ignoring all the fancy terms (REST, SOAP, GraphQL, ...) for now, at the end of the day, an API is there to serve one fundamental purpose:

> Defining a contract between a client and a server.

Nothing more, nothing *less*. By defining a contract, an API allows a client (end user, another service, etc) to **easily** communicate with the server (or service).

As such, your API shouldn't be just an extension of your service, it should represent a *simplified* and *generalized* extension of your internal system.

It shouldn't matter what language your backend is in or what questionable tech decisions you've made, as long as the API you offer is **good**.

### What is an API really?

A magical thing you don't maintain that someone you don't know wrote and you integrate with and fully trust and expect to work today, tomorrow and maybe a year from now.

An API is a trust exercise between two developers who never met.

## What makes an API accesible?

The protocol it uses.

Imagine you want to open up a world wide business but you have limited resources and can't afford to translate all your material to satisfy all customers in all countries. What do you do? You pick the most common language that's going to cover the largest percentage of your client demographic.

Same concept applies to APIs. Your goal as a *System Designer* working on an API, is to make it as accesible as possible to other developers and clients. As such, you should pick a common protocol for your target users.

Sure you could argue that any network connected machine in the world is capable of speaking TCP/UDP, but smarten up. HTTP(S) is the protocol you should be aiming for (this doesn't mean you have to be exlusive and can't offer your API over multiple protocols).

### Why HTTP?

HTTP fits so nicely in the entire paradigm of APIs (mainly REST). I won't go into much detail, but here is a list:

* Any self respecting language has a builtin or community library to support HTTP
* It's easy to secure with TLS
* Allows separation of concern by having the concept of "HTTP call" to one endpoint
  * One request, to one endpoint, leads to one response
* [HTTP status codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) have universal meanings and map nicely to any expected API behaviour
  * `200 OK` - all is good
  * `201 Created` - (not a fan of using this, but it works)
  * `204 No Content` - basically a soft/expected 404
  * `400 Bad Request` - read the docs and try again
  * `401 Unauthorized` - you forgot the API key dummy
  * `403 Forbidden` - this endpoint doesn't come with the free version of our API, pay up
  * `404 Not Found` - `¯\_(ツ)_/¯`
  * `5xx` - we ƒuçked up
  * `4xx` - you ƒuçked up
* A HTTP request has multiple components, each having a diffrent role
  * URL
    * Allows you to nicely group endpoint by concern
      * `/api/users/:user_id/`
      * `/api/users/:user_id/friends`
      * `/api/objects/:object_id`
      * `/api/admin/operation`
    * Allows the use of "params": `:user_id`, `:object_id` to pass "variables"
    * Query args for *filtering*
      * `?country=RO&age=24`
      * `?page=2&limit=50`
      * `?dateFrom=2020-01-01&dateTo=2020-06-01`
      * `?isAwesome=true`
  * Methods (following a REST paradigm)
    * `GET` - for READ operations
    * `POST` - for WRITE operations where a **new** resource is created
    * `PUT` - for replacing information of an object
    * `PATCH` - for updating information of an object
    * `DELETE` - for removing a resource
    * Don't forget that at the end of the day, HTTP methods are just a string, you can use whichever for whatever, it's a common misconception that HTTP methods hold any real value or functionality. For all a server cares, you can use `DELETE` to READ data and `GET` to create new entries
  * Body
    * The body of the requests you accept and responses you send is a big topic we'll cover.
  * Headers
    * A great place to put misc things, I always think of headers like a sidecar on a motorcycle
    * Spoiler alert, your `Authorization` header should go here!
    * Headers are all around great to have and even greater for passing metadata and *passive* information
    * Beware of the [*invisible limits*](https://stackoverflow.com/questions/686217/maximum-on-http-header-values)

### Protocol is just half the game

The language you speak is just half of it, the other half is about the message *format*.

In the case of APIs using HTTP, the request and response types are defined using media type headers (`Accept`, `Content-Type`).

In the case of modern APIs I'd say you're best to go with JSON. XML is acceptable if you are integrating with legacy systems or need to offer legacy support. Any other reason or format should not be tolerated.

Using YAML gets you the death penalty.

Using JSON or XML doesn't mean your API is good, there are plenty ways of doing poor JSON/XML responses/requests.


Imagine the following response:

```
[
  {
    "name": "Item 1",
    "cost": 31.40
  },
  {
    "name": "Item 2",
    "cost": "31.40$"
  },
  {
    "name": "Item 3",
    "cost": 3140
  }
]
```

This would require reflection and extra logic on the client side to handle the response.

In modern languages, the format is important as that is the layer that is going to map back and forth to your logic structs. In Go for example, you can map structs to JSON (or other formats) using "tags".

```go
Foo struct {
  Bar  int     `json:"bar"`
  Baz  *string `json:"baz,omitempty"`
  Soap bool    `json:"soap_bar"`
}
```

This can then map to and from:

```json
{
  "bar": 14,
  "baz": "hello world",
  "soap_bar": false
}
```

* [JSON to Go](https://mholt.github.io/json-to-go/)

## What makes an API usable?

Well, going back to the "usefullness" of an API 

> Defining a contract between a client and a server.

The usability aspect is defined by yet another fundamental concept. Why use an API in the first place?

> To solve a problem.

Usability is defined by how good of a tool an API for the job at hand. Offer as much functionality as possible behind an interface as simple as possible.

## What makes an API robust?

> Stateless, Consistency, Abstraction, Deterministic

It's hard to keep the four separated, as they sometimes intersect.

### Statelessness

Trying to keep your API as stateless as possible. You don't want endpoints to change behaviour based on "state" you set with previous calls.

Creating endpoints with dependencies on calling other endpoints is bad.

### Consistency

It doesn't matter if an API is consistently bad or consistently good as long as it is consistent.

There is nothing worse than unexpected behaviour and edge cases in an API.

It's very important with activly developed APIs to use versioning or to keep backwards compatability. (Versioning is good in the path and acceptable in a header).

### Deterministic

Making a request twice should give the same answer as long as the data behind the API did not change.

Making multiple requests to the same endpoint with the same data shouldn't change the response unless the data changes acordingly.

### Abstraction

An API should hide complexity from the user where apropriate.

As such an API should provide as few endpoints as possible. If you offfer 100 endpoints, but all users use the same 5 endpoints, then consider scaling down. Also if your users have to run 5 requets to get 1 job done, consider hiding the complexity of the 5 endpoints behind a single one.

Keep in mind, that it's your job and responsability as a System Designer to put in the extra work *now* in order to save others months of frustration and pain trying to integrate with your lazy API.

## What makes an API secure?

First concern should be around securing communications. In the case of HTTP, that is achieved by serving your API over HTTPS (with TLS). There is no reason to have your API served over **plaintext HTTP**.

Next, is the Auth* bit. For this I thought it would be nice to see diffrent aproaches, ranging from "I'll see you in hell" to "Thanks, that's how it should be done":

![See you in hell](https://media.giphy.com/media/uA6sERHUlCXYI/giphy.gif)

* Cookies - it's a pain to parse & pass, and multi-instance services must somehow share the cookie in order to not deauth each other
* Credentials in the path of the URL (WHY?!!)
* Passing username and password as query args - most proxies, routers, etc will have no problem with logging the URL or caching it somewhere with your secrets
* Credentials in the body of the request (STOP!)
* Using basic auth (ok, not the best, not the worst)
* Using any form of decent auth: (yes and thank you)
  * Bearer token
  * API key
  * JWT

Extra hell points for:

* IP whitelisting without CIDR support
* Using an undocumented header for Authorization
* Not using `401` and `403` status codes, instead giving some generic `4xx` or `5xx`
* Forced use of IP whitelisting
* User-Agent rate limit

Extra good points:

* High rate limit per IP
* Rate limit per API key

## What makes an API performant?

Performance is relative in the world of computing. When working with APIs over the internet speed is quesionable and it's important to know where to optimize.

You know you can take down response time by 0.5ms for one endpoint if you spend a week on it? Is it worth it? What about trimming 10ms or 50ms or 100ms from the response time? Where does it become worth it is up to you and your team.

![Latency](https://i.imgur.com/k0t1e.png)

It's also relative to the importance of your API. If your API servers "Cat of the day pictures", it's not critical to have 99.9% uptime and 5ms response times for users in Africa using 2G networks.

If your API is some stock market ticker, sure, have high availability, serve things over a distributed network of servers, use caching for popular requests so you don't read from disk every single time (check image above), squeeze out every last ms out of your response time.

There's plenty of crazy optimizations you can do, some might be sane for your use case some might be insane. Pick carefully.

* Optimize marshling and unmrashiling of JSON/XML
* Stop using padded formats and use raw bytes and well defined protocols
* Stop using TCP and send packages over UDP
* Reduce memory allocations and heap usage

### Caches

Crazy things aside, caches are a simple thing to do in most common APIs. Especailly important if your API integrates with another API behind the scenes. Instead of calling the 3rd party API everytime a user calls your API, you can instead cache the result (for a well selected TTL) and even give the user the option to bypass the cache and accept the higher response times. Everyone wins.

* The client gets faster responses and a choice
* Your API makes less calls and processing
* The 3rd party API spends less resources on your

## What makes an API pretty?

* Well defined documentation ([OpenAPI](https://www.openapis.org/))
  * An accepted standard for defining REST APIs
  * Easy to generate language specific clients or mock servers
  * Easy to generate HTTP API Client workspaces (for Postman, Insomnia, etc)
* Clear use of HTTP methods & status codes
* Consisntent error message format and explanations
* Providing libraries for certain languages
* **More documenetation**

## What makes an API ni̡̫̗̝̦ͅg̟͖̲̕ḩ̖̜̙͕t̘͞ṃ̷͎̥͉͍̦ͅe̩̱͚ͅr҉i̵̩̼̪̲̠̥s͘h̴͈̮ͅ?

* `518 This API doesn't work on weekends`
* Trimed responses
* Mixed formats
* Lack of error messages
* Keeping state on failed requests
  * Imagine making a POST request to create an object
  * The response comes back with an error
  * The object is created anyway but with bad values
  * Now you have to delete the object yourself before trying to re-create it
  * 10/10
* Global rate limit
* Mismatch between documentation and reality
* Multiple documentation files but none is accurate
* Breaking changes
