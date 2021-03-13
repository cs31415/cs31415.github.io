---
layout: post
category: blog
title: Open-Closed Principle
permalink: /blog/open-closed-principle
description: Organized Hacking Using the Open-Closed Principle (SOLID Principles)
image: ocp.jpg
---

![Open-Closed Principle](../../../img/ocp.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@jannerboy62?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Nick Fewings</a> on <a href="/s/photos/open-closed-sign?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></span>

The **Open-Closed Principle** states that software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification. 

> One way to describe the open-closed principle and the consequent object-oriented techniques is to think of them as a __organized hacking__.... The organized form of hacking will enable us to cater to the variants without affecting the consistency of the original version. 
>
> *-Bertrand Meyer*

It was originally conceived by [Bertrand Meyer](https://en.wikipedia.org/wiki/Bertrand_Meyer), French computer scientist and inventor of the [Eiffel](https://en.wikipedia.org/wiki/Eiffel_(programming_language)) programming language, in his book [Object Oriented Software Construction](https://www.amazon.com/Object-Oriented-Software-Construction-Book-CD-ROM/dp/0136291554) first published in 1988:

> Modules should be both open and closed. *-Bertrand Meyer*

This sounds like an oxymoron. *Open* means that the functionality of the module can be extended. *Closed* means that the source code of the module is no longer modifiable. Effectively, we should be able to extend a module's functionality without modifying *existing code*. 

What's wrong with modifying existing code? For one, it introduces the risk of breaking existing functionality. For another, the source code might not even be available for modification if the software is developed by a third party. 

How might we attain this haloed open-closed state? Meyer's answer back then was *inheritance*. By deriving a class from the original, one can extend its functionality by adding behavior or data. Inheritance as the mechanism of choice for adding posthoc functionality is no longer favored though, since it tightly couples derived classes to the base implementation, and inheritance hierarchies can get unwieldy.  

> It is possible for a module to manipulate an abstraction. *-Bob Martin*

[Bob Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) "extended" this principle in his [1996 article](https://drive.google.com/file/d/0BwhCYaYDn8EgN2M5MTkwM2EtNWFkZC00ZTI3LWFjZTUtNTFhZGZiYmUzODc1/view) to use **interfaces**, which he calls *abstract base classes* (sans implementation). In other words, classes can be open-closed by having them use interfaces instead of other classes. The interface itself is just a **contract specification** that says what features a class must provide with no regard to how they implement them. The open-ness comes from the ability to pass in any concrete class that adheres to the contract. A fancy word for such classes is **polymorphic**. The caller doesn't need to know the specific class. Indeed the specific class might not even exist until a future date. The caller thus no longer has a dependency on another class. This seemingly simple idea makes classes less fragile and easier to test and reason about. 

Some code examples will help illustrate this principle.

#### Non-conformance to OC Principle:

We have here a simple order processing class for an e-commerce store with a single `Purchase` method to complete the sale. In this example, any time a new payment method is added, the `OrderProcessor`'s `Purchase` method will need to be updated. Thus, `OrderProcessor` is not closed.

```javascript
class OrderProcessor
{
	public void Purchase(Cart cart)
	{
		switch (cart.PaymentMethod)
		{
			case PaymentMethod.CreditCard:
				var creditCardProcessor = new CreditCardProcessor();
				creditCardProcessor.ProcessPayment(cart);
				break;
			case PaymentMethod.Paypal:
				var venmoProcessor = new PaypalProcessor();
				venmoProcessor.ProcessPayment(cart);
				break;
		}
	}
}

class PaypalProcessor
{
	public void ProcessPayment(Cart cart)
	{
		// charge via Paypal
	}
}

class CreditCardProcessor
{
	public void ProcessPayment(Cart cart)
	{
		// charge the customer's credit card
	}
}

class Cart
{
	public Product[] Products { get; set; }
	public PaymentMethod PaymentMethod { get; set; }
}

class Product
{
	public string ProductId { get; set; }
	public string Description { get; set; }
	public double Price { get; set; }
}

enum PaymentMethod { CreditCard, Paypal }
```

#### Conformance to OC Principle:
In this example, the payment processor dependency is abstracted away through the `IPaymentProcessor` interface, with the specific payment processor being constructed by a factory class that is injected into `OrderProcessor`'s constructor. Thus, the `OrderProcessor` class is closed yet open to adding new payment methods without having any dependencies to specific payment processors.

```javascript
class OrderProcessor
{
	private readonly IPaymentProcessorFactory _paymentProcessorFactory;

	public OrderProcessor(IPaymentProcessorFactory paymentProcessorFactory)
	{
		_paymentProcessorFactory = paymentProcessorFactory;
	}
	public void Purchase(Cart cart)
	{
		var paymentProcessor = _paymentProcessorFactory.GetPaymentProcessor(cart.PaymentMethod);
		paymentProcessor.ProcessPayment(cart);
	}
}

internal interface IPaymentProcessorFactory
{
	IPaymentProcessor GetPaymentProcessor(PaymentMethod paymentMethod);
}

internal interface IPaymentProcessor
{
	void ProcessPayment(Cart cart);
}

internal class PaypalProcessor : IPaymentProcessor
{
	public void ProcessPayment(Cart cart)
	{
		// charge via Paypal
	}
}

internal class CreditCardProcessor : IPaymentProcessor
{
	public void ProcessPayment(Cart cart)
	{
		// charge the customer's credit card
	}
}

internal class Cart
{
	public Product[] Products { get; set; }
	public PaymentMethod PaymentMethod { get; set; }
}

internal class Product
{
	public string ProductId { get; set; }
	public string Description { get; set; }
	public double Price { get; set; }
}

public enum PaymentMethod { CreditCard, Paypal }
```

### Takeaways
- The Open-Closed principle says that software modules should be extensible (open) without requiring old code to be modified (closed).
- It enables old code to perform new tricks.
- It achieves this by injecting interfaces into the caller for any classes it needs to use rather than having the caller call the classes directly.
- This decouples the caller from dependencies on other concrete classes and makes it easier to reason about and test in isolation from the rest of the system.

### References
- [C++ Report - The Open-Closed Principle](https://drive.google.com/file/d/0BwhCYaYDn8EgN2M5MTkwM2EtNWFkZC00ZTI3LWFjZTUtNTFhZGZiYmUzODc1/view)
- [Clean Coder - The Open-Closed Principle](https://blog.cleancoder.com/uncle-bob/2014/05/12/TheOpenClosedPrinciple.html)
- [Uncle Bob SOLID principles Talk](https://www.youtube.com/watch?v=zHiWqnTWsn4)
