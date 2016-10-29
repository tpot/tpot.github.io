---
layout: post
title: A Pattern for Unit Testing Completion Blocks in Objective-C
categories: programming, objective-c
---

Asynchronous APIs turn out to be a very useful way of programming,
especially in event-based frameworks like Apple's iOS and OS X.  The
prevalent idiom for implementing asynchronous APIs is using <a
href="https://en.wikipedia.org/wiki/Blocks_(C_language_extension)">blocks</a>.
Blocks are an implementation, by Apple, to their C, C++ and
Objective-C compilers that implement closures.  Having functions being
first-class citizens via blocks gives the three aforementioned
languages a whole new flexibility, while retaining the language's
previous advantages.

Let's imagine we have a function that takes both a success block and a
failure block. I like to name functions of this form
<tt><i>something</i>OnSuccess:onFailure:</tt>.  The success block may
be called with some results as parameters, and the failure block is
called with an NSError instance.

{% highlight objc %}

// Do stuff and call blocks on success or failure

- (void)doStuffOnSuccess:(void (^)())successBlock
               onFailure:(void (^)(NSError *))failureBlock
	       
{% endhighlight %}

How can we unit test this function? Naively, we can go ahead and test
the different scenarios one by one like so.

{% highlight objc %}

// Test scenario A

- (void)testDoStuffScenarioA
{
    XCTestExpectation *expectation =
        [self expectationWithDescription:@"ScenarioA"];

    [sut doStuffOnSuccess:^{
    
        // Assert things here
	
        [expectation fulfill];
	
    } onFailure:^(NSError *error) {
    
        XCTAssertNotNil(error);
        XCTFail(@"%@", [error description]);
	
       [expectation fulfill];
    }];

    [self waitForExpectationWithTimeout:10 handler:^(NSError *error) {
        XCTAssertNil(error);  // Timeout or other error if non-nil
    }];
}

// Test scenario B

- (void)testDoStuffScenarioB
{
    // Pretty much an identical cut&paste of testDoStuffScenarioA
    // but with a few test-specific tweaks
}

{% endhighlight %}

This sort of thing works quite well for a few tests but as more tests
are added, especially when you want to test the failure block, it
quickly becomes unwieldy and the <a
href="http://wiki.c2.com/?DontRepeatYourself">DRY principle</a> is
violated repeatedly and flagrantly.  For tests that consider the
success block the failure block boilerplate code is duplicated, and
for tests that consider the failure block it's the success block.
Yuck.

Here's my solution.

Firstly we write a wrapper for calling the
<tt>doStuffOnSuccess:onFailure:</tt> method that bookends it with the
call to <tt>expectationWithDescription:</tt> and
<tt>waitForExpectationWithTimeout</tt> call.  This wrapper's single
responsibility is to manage the XCTestExpectation instance and allows
tests to be written with slightly fewer lines of code.

{% highlight objc %}

- (void)performDoStuffOnSuccess:(^())successBlock
                      onFailure:(void (^)(NSError *))failureBlock
{
    NSString *description = [NSString stringWithFormat:@"%s",
                                __FUNCTION__];
    
    XCTestExpectation *expectation =
        [self expectationWithDescription:description];

    [sut doStuffOnSuccess:^{

        successBlock();
        [expectation fulfill];
	
    } onFailure:^(NSError *error) {

        failureBlock(error);
        [expectation fulfill];
    }];

    [self waitForExpectationWithTimeout:10 handler:^(NSError *error) {
        XCTAssertNil(error);  // Timeout or other error if non-nil
    }];
}

{% endhighlight %}

Secondly we write  two helper functions that call  the wrapper.  These
helpers address the case when we test the success block, and the other
the failure block.   The boilerplate code for the unused  block is not
duplicated.

{% highlight objc %}
- (void)performDoStuffOnSuccess:(^())successBlock
{
    [self performDoStuffOnSuccess:^{
    
        successBlock();
	
    } onFailure:^(NSError *error)failureBlock {
    
        XCTAssertNotNil(error);
        XCTFail(@"%@", [error description]);
    }];
}

- (void)performDoStuffOnFailure:(^())failureBlock
{
    [self performDoStuffOnSuccess:^{
    
        XCTFail();
	
    } onFailure:^(NSError *error)failureBlock {

        failureBlock(error);
    }];
}

{% endhighlight %}

Now our test cases look like this:

{% highlight objc %}

// Test scenario A

- (void)testDoStuffScenarioA
{
    [self performDoStuffOnSuccess:^{
    
        // Assert things here
    
    }];
}

// Test scenario B

- (void)testDoStuffScenarioB
{
    [self performDoStuffOnFailure:^(NSError *error) {

        // Assert things here

    }];
}

{% endhighlight %}

I'm pretty happy with this as a pattern for testing completion blocks
and as an added bonus the method names and signatures have a pleasing
symmetry.