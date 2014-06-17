---
layout: post
title:  "Dependency Injection: Providers"
date:   2014-05-?? 23:29:54
categories: testing
---

In the last post I introduced Dependency Injection as a way to decouple a class from the specific implementation of its dependencies. This way, we could replace those dependencies with mocks in a unit test.

But there is a special kind of dependency: sometimes a class ``A`` needs to create an arbitrary number of instances of class ``B`` and then uses them. It is clear that ``A`` depends on ``B`` but… how can we decouple it so that ``A`` can be tested without using the actual instances of ``B``?

<!--more-->

One approach would be overriding/swizzling somehow the ``alloc`` method of ``B`` in our tests: we leave the original implementation for our production code, but we stub the alloc method to return our mock on tests.

However, even if we managed to make this work, we would have to face another problem: class ``B`` may have other dependencies. If this is the case… should ``A`` receive all the dependencies of ``B`` so that they can be injected into ``B``? That would mean making ``A`` depend on stuff that it doesn't need.

Therefore, if we discard replacing the implementation of ``alloc`` and we don't want to propagate our dependencies through our classes, we only can abstract object instantiation using _providers_.

A _provider_ is just an intermediate class whose purpose is instantiating objects. That means that it's also its duty to store and inject the dependencies of the created objects.



- Solution: intermediate class whose purpose is instantiating objects

- Example of a unit test that injects mocks through a mocked provider.

- The example could consist of an implementation of FVTwitterMessageSending that create a FVTwitterMessageSendCommand and executes it. If we don't mock the command, we would be sending actual messages. Therefore we use a provider, and we mock it so that it returns a mocked command.
