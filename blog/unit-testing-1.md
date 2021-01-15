---
layout: default
title: Unit Testing Part 1
description: What is unit testing and why do we need it?
image: unit-testing-testbytes.jpg
---
### {{ page.title }}

![Unit Testing](../../../img/unit-testing-testbytes.jpg)
<span class="credit">Image by <a href="https://pixabay.com/users/testbytes-1013799/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=762486">testbytes</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=762486">Pixabay</a></span>

### What is a unit test?

A Unit Test is test code that exercises a subset (unit) of functionality referred to as the [system under test (SUT)](https://en.wikipedia.org/wiki/System_under_test). e.g. one method of a class or a standalone function. 

### What to test?
There are two approaches according to [Fowler](https://martinfowler.com/articles/mocksArentStubs.html#ClassicalAndMockistTesting). The **mockist** and the **classicist** approach. According to the former, mock objects are used for both the SUT and dependencies.
This is more of a white box approach that requires knowledge of implementation details. The **classicist** approach is what Fowler favors, and is the focus of this article. This is a black box approach in which the real SUT object is instantiated and invoked and dependencies such as database, web service calls, environment-specific things like configuration, cookies etc. (which should ideally be injected into the SUT) are simulated. Just like a taster doesn't add sugar and milk when tasting tea, the idea is to only test the SUT without adding any external flavoring from dependencies. 
This is important for the following reasons:
- Dependencies don't color the results of the test
- Effort and time to setup dependencies (which might be recursive) is saved
- Tests can run faster which is important for developer productivity

### Why write unit tests?

- **Makes refactoring easier**
Tests encourage refactoring by providing confidence that changes haven't broken functionality
- **Ensure low-level system integrity**
Tests ensure that the building blocks of the system (the legos), taken individually, are functioning correctly. 
- **Help isolate bugs**
They make it easy to test a unit of code in isolation from the rest of the system. This is also faster since we avoid setup time to bring the system into the desired state for the test. 
- **Help exercise scenarios that are hard to test**
Abnormal scenarios such as database or network errors are hard to simulate during testing. Unit tests ensure that code behaves correctly in both good and bad scenarios.
- **Facilitates good design**
Testing provides feedback that helps us design more usable code. Test Driven Design (TDD) takes this to the extreme of writing tests before writing code. Thus if a test is hard to write, it is an indication that clients will find it hard to use. Also, writing tests forces us to think about the dependencies of a piece of code and thereby minimize undesirable coupling to those dependencies.
- **Improves system stability**
Adding new features should never break existing functionality, otherwise we risk destabilizing the system by introducing unknown side effects with every modification. Tests provide a guarantee against these kind of cascading errors. 
- **Speed up dev cycle**
Unit tests provide automated low-level regression testing, freeing QA time and creativity to concentrate on integration testing new features. 

### Test Doubles - Mocks, Fakes, Stubs, etc.

- Test doubles are **pretend objects** that stand in for the real object. They provide the same interface as the real object, and the caller is none the wiser. Dummies, stubs, spies, fakes and mocks are classifications made by [Gerard Meszaros](http://xunitpatterns.com/Test%20Double%20Patterns.html).
- **State vs behavior verification**
    - Verifying state of SUT (e.g. contents of a variable) after running test vs verifying an expected action (e.g. a database query or send email).
    - Mocks (and spies to a cruder extent) enable behavior verification by allowing us to setup and verify expectations on objects
- **A dummy** is supplied to fill parameter lists but never actually used. This would just be an empty implementation of an interface.
- **A stub** is the simplest pretend object that returns canned responses regardless of input passed. e.g. an HTTP client that always returns 200 or another fixed status code. **Spies** are stubs that provide rudimentary behavior verification by tracking calls made. e.g. how many times a particular method was called, what inputs were passed etc.
- **A fake** provides a simplified real implementation intended for testing. e.g. using a file instead of database for storage, returning different outputs based on inputs passed etc. This is what I was using before I knew about mocking frameworks. Typically it involves implementing a fake class that implements the same interface as the real class.
- **A mock** is the most sophisticated type of test double. It allows for behavior verification in addition to state verification. For example, we can verify that a certain method was called a certain number of times with certain input values, or that certain input values triggered a certain type of exception. With mocking frameworks such as Moq, we can setup complex expectations with a single line of code, without the need to create fake classes and implementations.
According to [Fowler](https://martinfowler.com/articles/mocksArentStubs.html#TheDifferenceBetweenMocksAndStubs), mocks are "objects pre-programmed with expectations which form a specification of the calls they are expected to receive".
