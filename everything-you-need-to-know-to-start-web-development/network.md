---
layout: default
title: Everything You Need To Know About Web Development Part 2
---

This is part 2 of a 4 part series I'm writing on all the concepts you need to
get started as a web developer. I'm going to try to explain everything
you need to get from here:

01010101011101000101011....

to here:

<img alt="Cat on keyboard" src="cat-on-keyboard1.jpg" style="width: 33%">

If you already know all about HTTP and transferring files over the network,
you can probably stop reading now and wait for part 3, where I'll try to explain
the DOM. If you skipped part 1, you can read it [here](encoding.html).

****

# Part 2: Transferring Files with the Hypertext Transport Protocol (HTTP)

In the last part of this series we discussed how you can represent nearly 
anything as a series of ones and zeros (bits), as long as you've agreed on a
protocol for interpreting them ahead of time. Now, let's assume there's a
mechanism to send a stream of bits between two computers and you want to copy
a file. You, once again, need to start by agreeing on a protocol to interpret
that stream of bits. In this article we'll mostly be explaining the Hypertext
Transport Protocol (HTTP), which is the most important protocol for web
developers to understand. 

If you want to get a copy of `filename` from the other computer, your first
instinct might be to send `get filename` in some appropriate character encoding
over the stream and have the other computer send back the bits of `filename` in
response. You wouldn't be too far off from what actually happens, but more on
that in a bit.

(We'll take for granted the protocols and conventions used by the operating 
system to communicate with the hardware to convert bits to something that
can be transported over a physical medium (radio waves, optical or electrical
cables, carrier pigeons, etc and just take for granted that someone's already
worked that out.  As far as we're concerned, we can just open a connection to
some other computer and get a bidirectional stream of data.)

Before we can even try sending `get filename`, we need a way to identify what
file on what computer we're talking about. The convention most people use for 
this is a specification called Universal Resource Locators (URLs) or Universal
Resource Indicators (URIs). There's probably some small technical difference
between the two terms, but most people use them interchangably, and the 
practical difference is that URL sounds old-school, while URI sounds pretentious.
Take your pick on which to use.

A URL usually looks something like this:
`http://driusan.github.io/everything-you-need-to-know-to-start-web-development/network.html`

This is composed of a number of parts:

`http` is the protocol to use to get the file. `://` is just a separator, as
far as we're concerned right now. `driusan.github.io` (everything until the
first slash after the `://`) is the hostname of the computer which has the file
and everything after that (`/everything-you-need-to-know-to-start-web-development/network.html`)
is the file that we want to get from `driusan.github.io`. If you're curious, you
can find the full technical specification for URIs by searching for a standard
called [RFC3986](http://tools.ietf.org/html/rfc3986).

So do we just open a connection to `driusan.github.io` and send
`get /everything[..]` using a UTF8 or maybe an ASCII character encoding? As I
said, it's close.

HTTP predates UTF8, and was designed in Switzerland, so the encoding is Latin1
(the standard character encoding standard in Switzerland at the time HTTP was
designed). Latin1 is close enough to ASCII or UTF8 that you probably don't need
to care about the difference unless you have accents or weird characters, but
it's a piece of trivia worth knowing. 

By convention, `GET` is capitalized, and you need to explicitly include the
version of the HTTP protocol you're using at the end of the request.

If you know how to use telnet or some tool on your OS well enough to directly
open a connection to `driusan.github.io` on port 80 (we're not going to talk
about ports today, but that's the standard HTTP port.) you can open a connection
and type:

```
GET /everything-you-need-to-know-to-start-web-development/network.html HTTP/1.0

```

And should have it dump the HTML for this web page in a response. (We'll get
into how to interpret that HTML in part 3.)

Note that there's an extra blank line after `HTTP/1.0`. That's because in
HTTP you can send an arbitrary number of key:value pairs with the request
called "headers" to send metadata to the server or modify the request in some
way. A blank line is the convention used by the HTTP protocol to indicate that
you're done sending the headers and want to get the response now.

There's a number of standard headers that are sent which each have their own
agreed upon meaning, but we're not going to have time to explain all of them
right now.

One that is worth explaining if you're playing along with telnet is `Host:`.
Notice that the `GET` request sent over the stream of data just had the filename
and the server responding to it has no obvious way of knowing what hostname the
request was for. If the server is hosting multiple domains and you sent the
above, the server would just take a guess which one you wanted and hope for the
best. The `Host` header is a standard header used to include this information,
so that the server can be sure it's sending the *right* data. The `Host` header
is required in HTTP/1.1, which is probably the most common protocol used on the
internet today. The above would look like this in HTTP/1.1:

```
GET /everything-you-need-to-know-to-start-web-development/network.html HTTP/1.1
Host: driusan.github.io

```

That's the minimal request that needs to be sent (for HTTP/1.1) to get the
server to send a valid response. Your web browser probably sends *many* more 
headers than that with each request (most web browsers have some kind of 
developers tools where you can see exactly what it's sending, and if you're
going to be a web developer it's worth learning how to use them, but this series
is about concepts, not tools.)

(HTTP/2 also exists, which is based on a binary encoding of the request, so it
isn't possible to directly open a connection and easily type a request on your
keyboard. The basic concepts in HTTP/2 are the same as in HTTP 1.x, but they
decided to use a binary encoding and use some other tricks to lower network
traffic at the expensive of human readability.)

So does the server just respond with the bits of the file? Almost, but it also
includes a little more metadata.

It starts with a line that has what's called an HTTP response code. It looks
something like this:

```
HTTP/1.1 200 OK
```

An HTTP response *always* includes a response code line as the first line. It
starts with the HTTP version (which should hopefully match what you sent with
the request), then a 3 digit number which indicates the status and a (fixed)
string representation of what that status code means in a human-readable way.

In this case, "200 OK" means "OK, everything's fine, here's your response." 
Hopefully "200 OK" is the most common response you get.

You can find a list of all the valid response codes pretty easily on the 
internet, but in general, 2xx is some variation of "Success", 3xx is some 
variation of redirection, 4xx is some variation of "the request is bad because
the client made a mistake" (including the famous "404 Not Found" error code),
and 5xx is some variation of "the request can't be fulfilled because something
went wrong on the server."

At a minimum, the server responds with a response code and then closes the
connection. Usually, however, it responds with a response code, then a series of
arbitrary key:value header pairs (just like the request), then a blank line, and
then the content of the resource.

So a complete request and response might look like this:

```
GET /hello.txt HTTP/1.1
Host: example.com

HTTP/1.1 200 OK
Content-Length: 5
Content-Type: text/plain; charset=UTF-8

hello
```

(where everything before the 200 OK line was sent by the client and everything
after it was sent by the server.)

The client asked for "hello.txt" using HTTP version 1.1. The server responded
with "OK", and also included 2 headers telling it that the file is 5 bytes long,
and told it what type of file it is, including what character set the file is
encoded with so the client knows how to interpret the response, then a blank
line denoting the end of the headers, and then sent the 5 bytes of file data
("hello")

## From Files To Applications

So far, we've only talked about transferring files, but that's mostly because
it's the easiest way to introduce the concepts. If transferring files were all
HTTP were good for we wouldn't have spent so long trying to understand it, and
we'd probably just use the File Transport Protocol (FTP) instead of the
Hypertext Transport Protocol (HTTP).

We started by talking about universal *resource* indicators, not universal
*file* indicators. So, what is a resource, anyways? It can be anything--all the 
client knows is that it asked for a resource, and got data back.
There's hints such as the above `Content-Type` header to help it decide how to
interpret the data, but the server could have done anything to generate it.
It could have read a file from a hard drive, or it could have calculated it
based on data in a database. The client doesn't know and shouldn't care.

What a resource represents is domain specific and where your job as a web
developer comes into play. Whether you're working in PHP or Go or Rust or Ruby, 
figuring out the business logic and deciding what URLs map to what resources,
how to encode them, and how the HTTP method (we'll get to that in a bit) from
the request affects it is the job of a backend web developer.

Are you writing a webmail client? Maybe a URL denotes an email, or another one
represents a mailbox. Writing a blog engine? Maybe a resource is an article, or
a comment. Are you writing a cloud services provider? Maybe a resource
represents the state of a VM or container.

So far, we've talked about how to `GET` a file, but that's not the only thing a
client can request to do to a resource, only the most common. What if you want
to place a resource on the server? Then you can send a `PUT` request. Delete it?
`DELETE`. The things that you can do to resources are called "HTTP methods", and
you can find the full list of which ones are valid and what they're supposed to
mean in (as well a the valid response codes and some standard header
definitions) in the HTTP/1.1 spec, [RFC2616](http://www.ietf.org/rfc/rfc2616.txt).

After `GET`, the second most common (and important) type of request is probably
a `POST` request, but to understand that we need to first explain a concept
called idempotency.

HTTP methods are defined as either idempotent or not. Idempotency is often 
explained as "not allowed to have any side effects", but that's not entirely
correct. Side effects like writing to a log file--which nearly every web server
does--are taken for granted. An idempotent method is one where, whether you make
the request one time, or a million times, the effect on the resource will be the
same.

Say you have a number in a database. If, in response to a request, you add one
to that number, it's not idempotent, because making the same request a second
time will have incremented it by two. Say you have that same number, and in
response to a request you set the value to 3. You can make that request once,
twice, or six hundred times, and the number will always be set to 3. (Maybe
there's another way to set it to 4, but *this* request always sets it to 3.)
That method is idempotent, because the end result doesn't depend on the number
of times you make the request.

`GET`, `PUT`, and `DELETE` are, by the definition in HTTP, idempotent requests
to get, put, and delete the identified resource respectively.

`POST` is not idempotent.  `POST` has a somewhat fuzzy definition of being
intended for things like "posting a message to a bulletin board, newsgroup,
mailing list, or similar group of articles" or "providing a block of data, such
as the result of submitting a form, to a data-handling process."

When you're posting a message to a forum (or "bulletin board", if you prefer)
you can't know the identifier of the resource ahead of time. How do you know the
identifier of your new message? It doesn't exist until after you've posted it,
which is why `POST` isn't idempotent, so the `POST` method exists in order
to give you a valid way to send things over HTTP that have side effects, like
creating a message, or processing user input from a form without violating
the standard.

Since it's not idempotent, if you post the same message twice, it's perfectly in
accordance with the spec to duplicate the message--though you probably want to
make life easier on your users by catching their mistake and warning them
instead. Most web browsers will popup a big warning if you try and refresh a
page that's the result of a `POST` request, so it's considered best practice
to take the request, do your processing, and respond with a redirect error code
instead of directly with a 200-series error code.

## Options

There's one final detail that didn't fit in anywhere else: URLs can
also include options. Options serve a similar purpose to request headers, but
are visible in the URL bar of the web browser. They're just key-value pairs
where the key and value are separated by equal signs, and the options are
separated by ampersands, starting from everything after the first ? in a URL.
For instance the URL: `http://www.example.com/hello?format=json&q=foo` denotes
the resource `/hello` on `www.example.com` with the options `format=json` and
`q=foo`. Another URL might be `http://www.example.com/hello?format=xml&q=bar`.
`/hello` is the same underlying resource, but this time you're sending a request
with an option saying that you want the format to be xml instead of json and
with the option of `q=bar` instead of `q=foo`, whatever that means.

## Summing Up

You can write your web application in whatever language you want and have it do
whatever you want, you really have 2 responsibilities:

1. Take a client's request to do something to a resource. Interpreting it is
   going to mean understanding the method, headers, and options sent and having
   a shared understanding with the client as to what they mean.
2. Send a response with an appropriate response code and data.

The rest is up to you and your imagination.

It's not hard to understand these concepts, but if you don't take the time to
learn them in the first place, it's easy to get them wrong, because no one's
going to be banging on your door complaining that you modified a resource with
a `GET` instead of `PUT`, or that your permission denied error page had a
`200 OK` response code instead of a `403 Forbidden`. For now, just trust me that
your life will be easier if you're strict with yourself about it.

****

So far we've only talked about the Hypertext Transport Protocol, but never
defined what hypertext means. Hypertext is just text which links to other text
using hyperlinks (more commonly just called "links" these days.) The most common
type of file to send over the Hypertext Transport Protocol (HTTP) is probably
Hypertext Markup Language (HTML) documents. Now that we have an idea as to how
to encode and transfer them, trying to understand how a web browser interprets
and displays them are what parts 3 and 4 of this series will be about.