---
layout: post
title:  "Node, REST, HTTP Standards, Semantic versioning and Javascript pain"
date:   2026-07-13 00:00:00 +0000
categories:
---
And so it was time for part 3 of FSO. While I wish I could progress even faster, rushing through this has little benefit if it means less understanding of the material. Although, as I continue to code, I realise how much of it requires practice, learned repetition, and exposure to new errors/concepts.

Alright, let’s take a look at the new theories introduced.

<br>
### What is Node?
FSO and Node themselves state Node as:
“Node.js® is a free, open-source, cross-platform JavaScript runtime environment that lets developers create servers, web apps, command line tools and scripts.”

That doesn’t really tell me much. But thankfully reddit users break it down perfectly in [this thread](https://www.reddit.com/r/learnjavascript/comments/3d4hs5/eli5_what_in_the_heck_is_nodejs/). The tldr is:
* Node lets you run Javascript code outside of your browser
* Node is able to run HTTP applications written in Javascript

Node uses `require()` to import modules. This is because back when Node.js was created back in 2009, Javascript had no native module system (import/export ES modules only standardized in ES2015). So Node adopted CommonJS, which uses `require()` to import modules. Node currently supports ES6 modules, but in FSO they’re sticking to CommonJS modules.

<br>
### Semantic Versioning
Semantic versioning is a way of naming versions numbers following the semantic versioning spec. It makes it easier to tell how much the new version has changed and if it’s backwards compatible.

| Code status                               | Stage         | Rule                                                               | Example version |
|-------------------------------------------|---------------|--------------------------------------------------------------------|-----------------|
| First release                             | New product   | Start with 1.0.0                                                   | 1.0.0           |
| Backward compatible, bug fixes            | Patch release | Increment the 3rd digit                                            | 1.0.1           |
| Backward compatible, new features         | Minor release | Increment the middle digit and reset last digit to zero            | 1.1.0           |
| Changes that break backward compatibility | Major release | Increment the first digit and reset middle and last digits to zero | 2.0.0           |

So what’s this? OwO
{% highlight json %}
"dependencies": {
  "my_dep": "^1.0.0",
  "another_dep": "~2.2.0"
}
{%endhighlight%}
The caret in front of `^1.0.0` means that if and when the dependencies of a project are updated, the version of Express that is installed is at least `1.0.0`. But the installed version can have a larger patch number (the last number) or larger minor number (the middle number). The major version (the first number) has to be the same.

`~` means that it only accepts patch releases (the last number).

<br>
### const variables in javascript and pain
Javascript is such a special language because ahahaha ooh my GOD what is happening here??
I briefly mentioned it once in [my previous post](../../../2026/07/07/useful-javascript.html) but my goodness, `const` not actually being an immutable constant is such a trip.

So how does it work?

In Javascript, a `const` is a constant reference to a value. Which means that it’s the memory address that the variable points to that cannot be changed. This is why you can modify attributes in a Javascript object but not re-assign it to a whole new object. The former is modifying the data in the same memory address, the other is re-pointing the variable to a different object in memory. This makes it much easier to remember and understand.

<br>
### About HTTP requests
Some useful acronyms I’ll run into often in web development and network architecture:

*REST* (Representational State Transfer) is an architectural style meant for building scalable web applications. 

*CRUD:* Create, read, update, and delete. The four basic operations of persistent storage.

There is also a request type `HEAD`, which does not return anything but status code and response headers (no response body).

The HTTP standard talks about two properties related to request types:
* *Safety:* Executing requests must not cause any side effects on the server (aka the GET request cannot cause any database changes). The response must only return data that already exists on the server.
* *All HTTP requests should be idempotent:* If a request does generate side effects, then the result should be the same regardless of how many times the request is sent. So HTTP PUT requests should result in the same thing no matter how many times the same PUT request is sent.

Both safety and idempotence are recommendations in the HTTP standard and not something that is guaranteed based on the request type.

<br>

[Previous Post](../../../2026/07/11/derived-data-states.html) | Next Post