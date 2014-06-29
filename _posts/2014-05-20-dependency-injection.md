---
layout: post
title:  "Dependency Injection"
date:   2014-05-20 23:29:54
categories: di testing engineering
---

Dependency Injection is not a complex concept.

When you use DI, your business logic classes don't instantiate the objects they need, and they don't access to singleton instances either. They ask in their constructors (preferably) for the objects they need.

<!--more-->

## The problem
Imagine a class that broadcasts messages (**FVBroadcaster**) to all friends of the current user. This system offers you a clear interface to send a message:

```objective-c
@interface FVBroadcaster : NSObject
- (void)broadcastMessage:(NSString *)aMessage;
@end
```

However, the way it sends the message to his friends is defined by the following protocol:

```objective-c
@protocol FVMessageSending <NSObject>
- (void)sendMessage:(NSString *)aMessage toUser:(FVUser *)user;
@end
```

We could have something like **FVEmailMessageSender** and **FVTwitterMessageSender**, which are different implementations that send the message to the e-mail address or the Twitter account of these users, respectively.

If we take a look to the whole code of **FVBroadcaster** we can see a **FVMessageSending** object as a dependency:

```objective-c
@interface FVBroadcaster : NSObject

// Dependencies
@property (nonatomic, strong) id<FVMessageSending> messageSending;

- (void)broadcastMessage:(NSString *)aMessage;

@end

@implementation FVBroadcaster

- (instancetype)init
{
    …
    // Twitter or e-mail?
    _messageSending = [[FV______MessageSender alloc] init];
    …
}

- (void)broadcastMessage:(NSString *)aMessage
{
    for each user…
        [self.messageSending sendMessage:aMessage toUser:user];
}

@end
```

So the problem here is… which implementation of **FVMessageSending** protocol should I instance inside **FVBroadcaster** initializer?

That's a tricky question, and the answer is… **none**!

If you hardcode the dependency inside the class, wouldn't you be violating the [Dependency Inversion Principle](http://www.objectmentor.com/resources/articles/dip.pdf)?

How could you write a unit test without sending e-mails or tweets?

```objective-c
- (void)testBroadcastMessageShouldSpamInTwitter
{
    FVBroadcaster *sut = [[FVBroadcaster alloc] init];

    [sut broadcastMessage:@"this is a test"];

    // Check your Twitter to see the messages sent to your friends… ?
}
```

It doesn't seem to be the way to go. What we should do is **ask** for the dependencies in the initializer and store them for later use:

```objective-c
@interface FVBroadcaster : NSObject

// Dependencies
@property (nonatomic, strong) id<FVMessageSending> messageSending;

- (instancetype)initWithMessageSending:(id<FVMessageSending>)messageSending;
- (void)broadcastMessage:(NSString *)aMessage;

@end

@implementation FVBroadcaster

- (instancetype)initWithMessageSending:(id<FVMessageSending>)messageSending
{
    …
    _messageSending = messageSending;
    …
}

@end
```
By doing this, your code is completely agnostic of the actual implementation of *messageSending*, and you can switch between both implementations when needed without modifying this class code.

Now, our unit test should be less spammy:

```objective-c
- (void)testBroadcastMessageShouldSendMessage
{
    id<FVMessageSending> mockMessageSending = mock(FVMessageSending);
    FVBroadcaster *sut = [[FVBroadcaster alloc] initWithMessageSending:mockMessageSending];

    [sut broadcastMessage:@"this is a test"];

    [[mockMessageSending shouldReceive] sendMessage:@"this is a test" toUser:any()];
}
```

Ignore the mock-related syntax, I just made it up. The point is, you can now easily replace your actual dependencies with mocks and test whatever you want.

But now the question is… who is going to provide **FVBroadcaster** with the correct instance in your production code?

## The solution
We need some external entity whose purpose is giving to our objects the collaborators they need. This is our Dependency Injection framework.

You can use an estabilished framework with lots of features (we use [Typhoon](https://github.com/typhoon-framework/Typhoon)) or just [do it yourself (DIY-DI)](http://misko.hevery.com/2010/05/26/do-it-yourself-dependency-injection/).

The responsibility of this framework is to wire **the whole object graph** by filling up all the dependencies that your objects need. This means your object graph is created upfront when your application starts (with some exceptions) and everything will be ready to be used later.

The ultimate goal of Dependency Injection is **separating the glue code** (code that wires up your objects) **from the business logic**. But it has another huge advantage: by making your dependencies explicit, **your code will be testable** (you can replace every dependency with a mock).

## Alternatives

Dependency Injection is just a solution to this problem. Another approach to decouple the actual implementation of collaborators is the [Service Locator pattern](http://en.wikipedia.org/wiki/Service_locator_pattern).

I personally prefer Dependency Injection: it is a simple concept and it forces you to make your dependencies explicit in your class interfaces.

## Additional benefits

Aside from improving testability, making your dependencies explicit makes it easier to spot classes violating the [Single Responsibility Principle](http://www.objectmentor.com/resources/articles/srp.pdf): a class with lots of dependencies surely has more than one responsibility. Take it as a [*code smell*](http://en.wikipedia.org/wiki/Code_smell).

## Conclusion

If you want to write testable code, you will surely need some way to replace your dependencies (like database, network, etc.) with mocks in your tests without modifying the code you want to test.

Among the available options, I recommend you to use Dependency Injection because it's a simple concept (it's also known as [*passing arguments*](https://twitter.com/slicknet/status/372798743948824576)) with additional benefits like helping you to fulfill the Single Responsibility Principle.

As an iOS developer, my personal choice is [Typhoon](https://github.com/typhoon-framework/Typhoon), which has served me well in Tuenti and Fever.
