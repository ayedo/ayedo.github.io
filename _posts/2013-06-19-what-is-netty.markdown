---
layout: post
title:  "What is Netty?"
date:   2013-06-19 17:54:07
categories: netty
---

Have you ever thought about creating your very own web server specific to your
needs in hope of achieving higher performance and throughput of your applications?
Do you sometimes dream of having more control over your networking code without
using the bumpy experience offered by bare metal networking primitives?

Then let me introduce to you to <a href="http://netty.io/">Netty</a>!

Netty is a networking library for Java: It was designed to support developers who
wish to create applications such as web servers, chat servers, message brokers
or whatever networking application you might come up with yourself.

There are mainly two things Netty could do for you:

- You'll probably find it easier to use and maintain than the standard Java ways
  of doing lower level networking I/O.

- Your application will be built on top of an architectural design with great
  potential for performance and scalability. In short: Netty will help you with
  writing and maintaining asynchronous code.

You might wonder what asynchronous means here? Since it the crucial aspect
behind Netty's architecture I would like to try and explain it to you first.

The word "asynchronous" sounds a bit scary at first but the concept is
actually likely to be known to most people from their daily life: It's all about
waiting.

Imagine for example that you are cooking spaghetti and you would also like to
have tomato sauce with it. The first thing you do is set up the water and then
you watch it for ten minutes until it boils. As soon as you see the water splashing
you put spaghetti in and start chopping onions and tomatoes for your sauce.

Wait, that was weird. Why would you watch the water until it boils and not just
chop your vegetables in the meantime? We only want to have to wait for something
to happen when there is absolutely nothing else to do and even then there is Reddit!

Have you ever had the feeling that a program you have just created is watching the
water until it boils and is refusing to do anything else in the meantime?
I clearly remember my first networking client-server application written for a
school assignment. I was using the standard Java 1.4 I/O libraries and it was a
lot of.... fun. It included something along the lines of:

<br>

{% highlight java %}
public void serve(int port) {
    // Create a socket for incoming client requests on the provided port
    final ServerSocket socket = new ServerSocket(port);
    while (true) {
    // Watch the server socket for new incoming connections.
    final Socket clientSocket = socket.accept();
    // <- It only continues here when a new connection has been accepted!
    ...
    // Since the thread is busy now, no new connections can be accepted
 }
{% endhighlight %}

<br>

In order to allow for new clients to connect my little program was watching out
for a new connection and would not be able to do anything else in the
meantime. Once it got busy with handling a request it would not accept any new
connections. New clients would thus basically told to... find themselves a
different server!

<br>
<br>
<center><img src="/img/blocking.png" width="70%"/></center>
<br>
<br>

Well, well you might say: There was a lesson for you to learn in that course.
Here, rookie engineer, you fix that problem by creating a thread for every request:

<br>
<br>
<center><img src="/img/blocking_threads.png" width="72%"/></center>
<br>
<br>

And suddenly it works! Our server now knows how to accept and handle requests "in the same time".

There is a twist to that solution though. If we go back to our kitchen example, that would
mean that I can always just call up a friend to chop my onions while I watch the water
until it boils. If we also want to have the tomatoes ready on time for the sauce,
we just need to get more friends. And since everybody knows you have crazy ways of cooking, they
willingly comply. Let's not forget: We now have to we make sure that all our friends
are on time with their part of the cooking. We should further try to not use stove at the
same time! We'll need to come up with some rules to avoid stepping on each others
feet as well... cooking spaghetti just got really complicated!

Making a kitchen work this way is hard and it turns out that many people find
multi-threaded programming complicated too. Threads are also quiet expensive in terms
of memory consumption and if you want to serve a lot of requests you might see your friends
faces pushed against the kitchen window gasping for air! Don't get me wrong: The concept of
using a thread per request is valid and has some useful properties.
But wouldn't it be nice if we could do I/O using a single or a small number of threads?
How could we do I/O with fewer threads and only wait when there are no events from any
of our input or output streams to react to?

Here is an idea:

Instead of being busy watching what one stream is doing while ignoring events on any other streams,
we could watch out for events on all of our streams combined! With such a mechanism, a single thread
could check up on all streams at once. It would only have to wait when no stream was offering
events for it to react to. This means that somehow we will have to provide a single thread
with the buffered events from all streams one by one:

<br>
<br>
<center><img src="/img/selector.png" width="72%"/></center>
<br>
<br>

When the developers at Sun decided to complement their blocking or synchronous I/O API (the one mentioned in the school assignment),
they ultimately used this idea as the basis for their solution: The Java New IO API which is often abbreviated to "NIO".

There are other ways of doing asynchronous I/O for which I do not have
the time to go into details here but I would like to mention that Java NIO later
got an extension called "NIO2" which allows you do have asynchrony using a different
concept than the "selector" approach mentioned above (it is based on
<a href="http://en.wikipedia.org/wiki/Callback_%28computer_programming%29">callbacks</a>).

What does Netty have to do with all of this?

For starters, it provides you with a unified asynchronous API abstracting over
the native Java I/O frameworks mentioned before.

Having a unified asynchronous API means that you can write your application against the
Netty API and tell it which underlying native API it should use for sending the bytes
over the network. You thus don't have to worry about many of the lower-level details
and problems that come with handling Java blocking IO, NIO or NIO2 code directly.
The interested reader might have noticed that I have also listed the blocking
Java IO as a possible means of transportation. This is correct! The Netty
developers pull some tricks that make it possible for you to introduce asynchrony
into legacy code by running on the old blocking I/O API.

Another problem, which is approached by Netty, is the complexity of asynchronous
code. Let's go back to our "selector" solution for achieving asynchronous I/O:
The Selector provides your single thread with event occurring on a group of streams.
This means that when your thread asks the selector for events you will get one out
of a whole possible mix of events (, such as "stream ready for writing, reading" etc...) and you'll have to come up with some structured way of figuring out how to react
to each of them.

One way of attaching logic to specific events is to use a large switch statement
with a "case" for each of the differing events you'd expect from your streams.
Typically, this approach leads to long methods with deep nesting. It is considered
by many developers to contain typical traits of unmaintainable code. The following idea
provides an alternative solution:

<br>
<br>
<center><img src="/img/reactor.png" width="72%"/></center>
<br>
<br>

We could implement a mechanism which takes the events from the selector
and distributes or "dispatches" them to interested methods.
If you would like react to an event you can thus subscribe a method to it.
Once the event occurs the dispatcher then calls the subscribed methods
(often called "handlers") one by one and one after another. In the figure it can be
seen that a "blue" event has occurred and the dispatcher now calls the subscribed
blue "event handlers" sequentially. This pattern was named the "Reactor" by it's
discoverers (<a href="http://www.cs.wustl.edu/~schmidt/PDF/reactor-siemens.pdf">link</a>) but it's also commonly known as an "event dispatcher"
or "the event loop".

Not only does Netty implement this reactor pattern for you
but it further extends it with the concepts of pipelines: Instead of having a
single handler attached to an event it allows you to attach a whole sequence of handlers
which can pass their output as input to the next handler. How is this useful? Usually,
networking applications work in layers: You have raw byte streams which represent HTTP
which might me used to transport compressed and encrypted data and so on. With Netty
you can now chain a HTTP decoder which passes its output to a decompression handler
which passes its output to an decrypt handler and so on... In short: Event handlers
in Netty are composable and composition is at the very core of most maintainable
and extendable code.

I would like to go a bit more into the scalability claims I have made about Netty and not
yet argued about.

A part of the mentioned scalability of Netty is a direct consequence of its asynchronous
design: It does not require a thread per request and is therefore able to handle more
concurrent connections with less available memory compared to a thread-per-request approach.
With less threads running on your server the operation
system will be less busy doing context switches and other thread related overhead.
This can lead to a performance increase. In the case of Netty this seems to be
true as it has found its way into businesses such as Twitter and Facebook which handle
impressive amounts of concurrent requests.

The rest of the scalability comes from the great expertise of the Netty developers in
Java and the specific performance bottlenecks in the native I/O
implementations. If you are interested in high performance data structures optimized
for network applications or if you would like to know more about how Netty handles threading
internally I recommend the book "Netty in Action". It is written by one of the main
contributors of Netty: Norman Maurer. At time of writing of this post the book is not yet
finished but it already gives the most complete overview on Netty in my opinion.

I hope this post could give you some insights on Netty and motivate you to write that
high-performance scalable networking you always had in mind! Happy hacking :)
