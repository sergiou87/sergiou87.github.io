---
layout: post
title:  "Dependency Injection: Providers"
date:   2014-06-29 21:00:54
categories: programming testing ios
---

In the last post I introduced Dependency Injection as a way to decouple a class from the specific implementation of its dependencies. This way, we could replace those dependencies with mocks in a unit test.

But there is a special kind of dependency: sometimes a class ``A`` needs to create an arbitrary number of instances of class ``B`` and then uses them. It is clear that ``A`` depends on ``B`` but… how can we decouple it so that ``A`` can be tested without using the actual instances of ``B``?

<!--more-->

One approach would be overriding/swizzling somehow the ``alloc`` method of ``B`` in our tests: we leave the original implementation for our production code, but we stub the alloc method to return our mock on tests.

However, even if we managed to make this work, we would have to face another problem: class ``B`` may have other dependencies. If this is the case… should ``A`` receive all the dependencies of ``B`` so that they can be injected into ``B``? That would mean making ``A`` depend on stuff that it doesn't need, and therefore couples ``A`` to ``B``'s implementation.

![A depends on B's dependencies](/public/dependency-injection-providers/ProvidersBAD.png)

Therefore, if we discard replacing the implementation of ``alloc`` and we don't want to propagate our dependencies through our classes, we only can abstract object instantiation using _providers_.

A _provider_ is just an intermediate class whose purpose is instantiating objects. That means that it's also its duty to store and inject the dependencies of the created objects.

![A depends on B's provider](/public/dependency-injection-providers/ProvidersGOOD.png)

## Example

Continuing with the example of the previous post, imagine our ``SPTwitterMessageSender`` implementation relays on some ``SPTwitterMessageSendCommand`` which actually does the sending work.

Whenever we want to tweet a message, our _sender_ will have to instance a _command_ and execute it. These command instances will require access to the Twitter client in order to tweet the desired messages.

```objective-c
@interface SPTwitterMessageSender <SPMessageSending>

// Dependencies
@property (nonatomic, strong) TWTwitterClient *twitterClient;

@end

@implementation SPTwitterMessageSender

- (void)sendMessage:(NSString *)aMessage toUser:(FVUser *)user
{
    SPTwitterMessageSendCommand *command = [[SPTwitterMessageSendCommand alloc] initWithTwitterClient:self.twitterClient];

    [command execute];
}

@end
```

This makes ``SPTwitterMessageSender`` depend on ``TWTwitterClient`` just for injecting it in the command instances. If we wanted to change ``SPTwitterMessageSendCommand`` to relay on some custom ``SPMyOwnTwitterClient`` we would be forced to change ``SPTwitterMessageSender``, which makes no sense.

Let's see how to implement a _provider_ for ``SPTwitterMessageSendCommand``:

```objective-c
@interface SPTwitterMessageSendCommandProvider

    // Dependencies
    @property (nonatomic, strong) TWTwitterClient *twitterClient;

    - (SPTwitterMessageSendCommand *)command;

@end

@implementation SPTwitterMessageSendCommandProvider

- (SPTwitterMessageSendCommand *)command
{
    return [[SPTwitterMessageSendCommand alloc] initWithTwitterClient:self.twitterClient];
}

@end
```

It's as simple as that.

Now let's see how it's used in ``SPTwitterMessageSender``:

```objective-c
@interface SPTwitterMessageSender <SPMessageSending>

// Dependencies
@property (nonatomic, strong) SPTwitterMessageSendCommandProvider *twitterMessageSendCommandProvider;

@end

@implementation SPTwitterMessageSender

- (void)sendMessage:(NSString *)aMessage toUser:(FVUser *)user
{
    SPTwitterMessageSendCommand *command = [self.twitterMessageSendCommandProvider command];

    [command execute];
}

@end
```

Now ``SPTwitterMessageSender`` depends only on the _provider_ and not on ``SPTwitterMessageSendCommand`` dependencies.

If we want to test if a command is executed, we can write it like this:

```objective-c
- (void)testSendMessageShouldExecuteCommand
{
    // Given
    SPTwitterMessageSendCommand *mockCommand = mock(SPTwitterMessageSendCommand);
    SPTwitterMessageSendCommandProvider *mockProvider = mock(SPTwitterMessageSendCommandProvider);

    [[[mockProvider shouldReceive] command] andReturn:mockCommand];

    SPTwitterMessageSender *sut = [[SPTwitterMessageSender alloc] initWithTwitterMessageSendCommandProvider:mockProvider];

    // When
    [sut sendMessage:@"this is a test" toUser:someUser];

    // Then
    [[mockCommand shouldReceive] execute];
}
```

As you can see, we have fixed both problems: we can easily replace our class with a mock and our SUT doesn't depend on our class' dependencies, just on the provider.

## Providers with Typhoon

My friend and code wizard [Daniel Rodríguez Troitiño](https://github.com/drodriguez) developed, while we were working at [Tuenti](http://www.tuenti.com), a very useful addition to [Typhoon](http://www.typhoonframework.org/): the ```TyphoonFactoryProvider```.

This class uses the runtime toolbox to implement these providers from the assembly file used by Typhoon, you don't need to manually inject the dependencies nor write the implementation of the factory method.

You only need to provide a protocol (the interface of your provider, with the factory method and the dependencies) and a block with the implementation of the factory method. That's all!

## Conclusion

Object instantiation can easily become a [seam](http://c2.com/cgi/wiki?SoftwareSeam) by just creating an object _provider_ whose only purpose is allocating the actual object.

Combining these providers with *dependency injection* (AKA making your dependencies explicit), it's really straightforward to replace that provider (and the provided object) in a test while keeping our classes loosely coupled.
