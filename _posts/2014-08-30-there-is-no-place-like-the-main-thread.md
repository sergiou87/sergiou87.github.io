---
layout: post
title:  "There's no place like the main thread"
date:   2014-08-30 00:50:54
categories: programming
---

In my experience as iOS developer, I've seen some inclination to do lots of things in separate threads just for the sake of performance and to avoid blocking the UI thread.

Stuff like blocks, [GCD](http://en.wikipedia.org/wiki/Grand_Central_Dispatch) and [NSOperations](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/NSOperation_class/Reference/Reference.html) seem really neat for this kind of tasks and they're super easy to use, but [with great power comes great responsibility](http://en.wikipedia.org/wiki/Uncle_Ben#.22With_great_power_comes_great_responsibility.22).

<!--more-->

Truth is, at least in my experience, that you rarely need to create separate threads to do anything, unless you're working with low-level stuff where only a synchronous API is available (like using [NSFileManager](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSFileManager_Class/Reference/Reference.html) to read a huge file). Otherwise, most of the work you do can be done in the main thread and you won't notice, your application will continue to run **smoothly**. And if it doesn't, then move that work to a separate thread. Remember: [premature optimization is the root of all evil](tttp://c2.com/cgi/wiki?PrematureOptimization).

Now imagine you actually need to perform some task in a background thread… how should you proceed? If they're some simple tasks (1-2 lines) that can just be dispatched with **GCD** go for it. If they're more complex (maybe a composition of several simple tasks), you better encapsulate that in a proper class (someone said **NSOperation** subclass?).

## Encapsulating your tasks

Regarding **NSOperation**s… I've seen some situations in which they're abused too. They're fine to encapsulate complex and heavy tasks in a class need to be run in a separate thread, and [NSOperationQueue](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/NSOperationQueue_class/Reference/Reference.html) provides a really easy way to manage concurrency and so on. But there's some times when the APIs you use to perform those tasks are managing background threads themselves.

For example, imagine you're using [AFNetworking](https://github.com/AFNetworking/AFNetworking) and you need an atomic task that downloads some data from some server, filters it somehow (but very quickly), and then sends it back to another server. Assuming the filtering process is really quick, and knowing that **AFNetworking** calls are already asynchronous and provide a beautiful completion block, you don't need to put all that code in a separate thread! Only the requests have to be made in another thread, and **AFNetworking** is already taking care of that for you. Therefore you don't need a **NSOperation**, you can work with a regular **NSObject**.

For **Fever** app I created my own simple system for queuing this **NSObject**-based operations, just a bunch of lines, and with lots of benefits: simple code, simple behavior, ridiculously low number of threads (and therefore lower resource usage) and an awesome performance.

An alternative to this would be creating a concurrent **NSOperation** subclass. That is, subclass **NSOperation** and override the method `isConcurrent` to return `YES`. But if you don't need cancelling, controlling the concurrency level and so on, I wouldn't recommend you to enter in that little hell of `isFinished`, `isExecuting`, `isCancelled`, etc. I don't have good memories about that either :P

## Don't forget to call me back

Most of the times, when you perform some asynchronous task you'll want to get a result or at least a chance to do something when that task has finished.

My short experience has shown me that it's very tempting to just notify when an asynchronous task has finished in the same background thread, and let the receiver of that callback switch to another thread if required. Why should you choose in which thread the callback should be run? Well, the answer is: for sanity. Working like this has led to some really chaotic situations with lots of threads executing some callback code simultaneously. Do you know what that mean? **Exactly: crashes really hard to debug and fix.**

While I was designing the architecture of **Fever** app, I took a decision that has saved me lots of headaches (and crashes): **every class doing asynchronous work assumes that it's used from the main thread, and all its callbacks/completion blocks will be invoked in the main thread when the task is finished.**

That may be obvious, but it's something that hasn't been done in previous teams I've worked with, and it was really painful in the long term. Fortunately, lots of third-party libraries are designed like this, making this decision easy to carry out.

## Conclusion

- Don't be afraid of running some code in the main thread: keep concurrency at a minimum.
- If you actually need some concurrency, take decisions to make your design simple (and your life easy): rely on the main thread to invoke asynchronous tasks and their callbacks.
- Avoid premature optimization at all costs.

