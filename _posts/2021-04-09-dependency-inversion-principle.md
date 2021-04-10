---
layout: post
category: blog
title: Dependency Inversion Principle
permalink: /blog/dependency-inversion-principle
description: Dependency Inversion Principle (SOLID Principles)
image: dip.jpg
---

![Dependency Inversion Principle](../../../img/dip.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@processingly?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">processingly</a> on <a href="https://unsplash.com/s/photos/invert?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></span>

Bad design exhibits rigidity (cascading changes), fragility (breakages in unrelated parts), immobility (reusing parts of the design in other apps is hard). 
[Bob Martin](https://en.wikipedia.org/wiki/Robert_C._Martin):
> What is it that makes a design rigid, fragile and immobile? It is the interdependence of the modules within that design.<sup id="cite1">[1](https://web.archive.org/web/20110714224327/http://www.objectmentor.com/resources/articles/dip.pdf)</sup>

How to improve a bad design that suffers from these flaws? That is the purpose of the Dependency Inversion Principle (DIP) which states that:
> A. High level modules should not depend upon low level modules. Both should depend upon abstractions.

> B. Abstractions should not depend upon details. Details should depend upon abstractions.<sup>[1](https://web.archive.org/web/20110714224327/http://www.objectmentor.com/resources/articles/dip.pdf)</sup>

### Dependency Injection vs inversion
Injection refers to passing in the dependency instead of instantiating it within the client. This alone isn't enough. The dependency passed must be an abstraction (interface or abstract base class), not the concrete implementation.

Instead of having the client create the concrete dependency, the dependency is created by a [factory method](https://en.wikipedia.org/wiki/Factory_method_pattern) or [IoC container](https://martinfowler.com/articles/injection.html) and an interface/abstraction (constructor or setter injection) is injected into the client. The client has no dependency on concrete implementations, only on the abstract interface. The implementing class itself has a dependency on the interface, which in accordance with [ISP](/blog/interface-segregation-principle), is tailored to the client. Thus, the dependency is inverted and points from dependent class to caller.

### Program to an interface, not an implementation

Dependency Inversion Principle appears to be identical to the following principle from the [GoF Design Patterns](https://en.wikipedia.org/wiki/Design_Patterns) book:
> Program to an interface, not an implementation.<sup id="cite4">[4](https://www.artima.com/articles/design-principles-from-design-patterns)</sup>

The point of both is the same. To minimize cohesion between classes. The fewer dependencies between classes in a large system, the easier it is to extend and test the system, and less the chance that changes in one module breaks the system in other places. 

### Examples:

The same examples used to describe [Open-closed principle](/blog/open-closed-principle) can be used here with some small modifications:

- #### Violation of DIP:

`OrderProcessor` has a dependency on the concrete implementations of credit card processor (for simplicity, we assume this site only supports credit card payments).  

```csharp
class OrderProcessor
{
	public void Purchase(Cart cart)
	{
		var creditCardProcessor = new CreditCardProcessor();
		creditCardProcessor.ProcessPayment(cart);
	}
}
```

- #### Conformance to DIP:

Payment processor abstraction is injected into `OrderProcessor`. Thus, `OrderProcessor` no longer has a dependence on the concrete implementation of credit card processor. Rather, the `CreditCardProcessor` now has a dependence on the `IPaymentProcessor` interface used by the `OrderProcessor` (since it has to implement it, and the interface in turn is tailored to the `OrderProcessor` in accordance with [ISP](/blog/interface-segregation-principle)) and in a sense the dependency is reversed.

```csharp
class OrderProcessor
{
	private readonly IPaymentProcessor _paymentProcessor;

	public OrderProcessor(IPaymentProcessor paymentProcessor)
	{
		_paymentProcessor = paymentProcessor;
	}
	public void Purchase(Cart cart)
	{
		_paymentProcessor.ProcessPayment(cart);
	}
}

internal interface IPaymentProcessor
{
	void ProcessPayment(Cart cart);
}

internal class CreditCardProcessor : IPaymentProcessor
{
	public void ProcessPayment(Cart cart)
	{
		// charge the customer's credit card
	}
}
```

### Dependency Inversion and unit testing

A nice side-effect of Dependency inversion using interfaces is that it enables writing unit tests without the need to setup complex dependency chains. Since external dependencies aren't part of the SUT (System under Test), their behavior (as specified in the interface) can be mocked using a tool like Moq, and unit tests don't need to have any cascading dependencies on concrete classes. See [Unit Testing Part 3](http://chandrasivaraman.com/blog/unit-testing-3/).

This completes our brief tour of [SOLID Principles](/blog/solid-principles).

### Takeaways

- Dependency Inversion Principle states that modules using other modules should depend on abstractions or contracts of those modules, not their concrete implementations.
- This reduces dependencies between modules, and enables modules to evolve independently of each other.
- In turn, this makes software less fragile and rigid, and makes it easy to reuse common code across systems without worrying about dependency chains.   
- DIP is a restatement of the GoF principle of coding to an interface, not an implementation.

### References

1. <a href="#cite1">[^]</a> [The Dependency Inversion Principle, Robert C. Martin, C++ Report](https://web.archive.org/web/20110714224327/http://www.objectmentor.com/resources/articles/dip.pdf)
2. [Dependency Injection by Hand](https://github.com/ninject/Ninject/wiki/Dependency-Injection-By-Hand)
3. [A Little Architecture](https://blog.cleancoder.com/uncle-bob/2016/01/04/ALittleArchitecture.html)
4. <a href="#cite4">[^]</a> [GoF Principles](https://www.artima.com/articles/design-principles-from-design-patterns)