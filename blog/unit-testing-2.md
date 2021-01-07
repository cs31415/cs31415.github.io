---
layout: default
title: Unit Testing Part 2 
description: What does a Unit Test look like?
---
### {{ page.title }}

![Unit Testing](../../../img/unit-test.jpg)

### What does a unit test look like?

So, in [part 1](/blog/unit-testing-1), we had an overview of what unit tests are and why they matter. [Abstraction](/blog/abstraction) is a wonderful thing, but meaningless without something concrete to give it context. Without further ado, here's what a unit test looks like (using C# and .NET Core):

```C#
using Xunit;

namespace UnitTestSamples.Tests
{
    public class MultiMapTests
    {
        [Fact]
        public void Add_KeyDoesNotExist_AddsValueToNewListForKey()
        {
            // Arrange
            var multiMap = new MultiMap<string, string>();

            // Act
            var key = "yellow";
            var value = "Sun";
            multiMap.Add(key, value);

            // Assert
            Assert.True(multiMap.ContainsKey(key) && multiMap[key].Contains(value));
        }
    }
}
```

The method `Add_KeyDoesNotExist_AddsValueToNewListForKey` on class `MultiMapTests` here is the unit test. We're using an (excellent) unit test library called [xUnit](https://xunit.net/) to run the test. The `Fact` attribute on the method indicates a test for invariant conditions. 

We use a standard pattern here to structure the unit test called [Arrange-Act-Assert](http://wiki.c2.com/?ArrangeActAssert). _Arrange_ is where we do any setup of initial conditions required for the test. _Act_ is where we exercise the SUT (System Under Test), which in this case is the `Add` method on the `MultiMap` class. _Assert_ is where we examine the aftermath to determine whether the test succeeded or failed.

There is also another pattern used here to name the test. It is called _Action-Scenario-Result_. The `Add` in the test name is the _action_ under test. `KeyDoesNotExist` describes the _scenario_ being tested and `AddsValueToNewListForKey` is the expected _result_. In short, we're testing whether calling the `Add` method with a non-existent key creates a new list with the value and adds it to the multimap.

Another type of test called a _Theory_ is a parameterized test that can be used for varying conditions. The same test can be used for multiple input conditions (the `InlineData` attribute whose values correspond to the parameters of the test method).
Since we add only the `"yellow"` key to the map, that condition returns true while the non-existent `"red"` key returns false.

```C#
[Theory]
    [InlineData("yellow", true)]
    [InlineData("red", false)]
    public void ContainsKey_VariousKeys_ReturnsStatus(string key, bool expectedStatus)
    {
        // Arrange
        var multiMap = new MultiMap<string, string>();

        // Act
        multiMap.Add("yellow", "Sun");

        // Assert
        Assert.True(multiMap.ContainsKey(key) == expectedStatus);
    }
```

As you can see, there is nothing terribly complex about writing unit tests. We'll get into more involved tests that use mock objects in a future post, but even there, the idea is to keep the test as simple as possible. If our tests turn out to be overly complicated to setup, that's a sign that the System Under Test (SUT) could use some refactoring.

#### Takeaways
- [_xUnit_](https://xunit.net/) is a unit testing library for C# and the .NET framework
- _Fact_ is a type of xUnit test for invariant conditions
- _Theory_ is a type of parameterized xUnit test for varying conditions
- _Action-Scenario-Response (ASR)_ is a useful naming standard for unit tests
- _Arrange-Act-Assert (AAA)_ is a way to structure the code for a unit test
