---
layout: post
title:  "Dependency Injection: Providers"
date:   2014-05-?? 23:29:54
categories: testing
---

In the last post I introduced Dependency Injection as a way to decouple a class from the specific implementation of its dependencies. This way, we could replace those dependencies with mocks in a unit test.

But there is a special kind of dependency: sometimes a class ``A`` needs to create an arbitrary number of instances of class ``B`` and then uses them. It is clear that ``A`` depends on ``B`` but… how can we decouple it so that ``A`` can be tested without using the actual instances of ``B``?

<!--more-->

- Problem: we cannot (should/must not) override ``alloc`` implementation so that we can replace the created instances with mocks
- Another problem: if ``B`` has dependencies, ``A`` should also have them so that they can be passed to ``B``… this doesn't scale!!!!

- Solution: intermediate class whose purpose is instantiating objects

- Example of a unit test that injects mocks through a mocked provider.

- The example could consist of an implementation of FVTwitterMessageSending that create a FVTwitterMessageSendCommand and executes it. If we don't mock the command, we would be sending actual messages. Therefore we use a provider, and we mock it so that it returns a mocked command.
