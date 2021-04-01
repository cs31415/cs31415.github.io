---
layout: post
category: blog
title: Interface Segregation Principle
permalink: /blog/interface-segregation-principle
description: Interface Segregation Principle (SOLID Principles)
image: isp.jpg
---

![Interface Segregation Principle](../../../img/isp.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@pawel_czerwinski?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Paweł Czerwiński</a> on <a href="https://unsplash.com/s/photos/waste-management?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></span>

The Interface Segregation Principle (ISP), another [Bob Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) creation, says:

**No client should be forced to depend on methods it does not use.**

Martin advocates splitting fat/general-purpose interfaces into thin/client-specific ones (what Martin Fowler calls `role interfaces`).<sup>[3](https://martinfowler.com/bliki/RoleInterface.html)</sup> The idea is to **reduce the dependency surface area** between object and caller to the minimum required. 

### ISP vs SRP
Some argue that ISP is just SRP([Single Responsibility Principle](/blog/single-responsibility-principle)) applied to interfaces<sup id="cite8">[8](https://blog.ploeh.dk/2009/09/29/SOLIDorCOLDS/)</sup>. 
Bob Martin himself clarified the difference between ISP and SRP in a [tweet](https://twitter.com/unclebobmartin/status/996739060348653568):
> ISP can be seen as similar to SRP for interfaces; but it is more than that. ISP generalizes into: “Don’t depend on more than you need.” SRP generalizes to “Gather together things that change for the same reasons and at the same times.”

ISP, thus, is a somewhat stoic principle, that urges consumers (objects, screens, web pages) to practise minimalism, to pare down their dependencies to the bare minimum needed to function. Providers correspondingly are urged to expose interfaces that are tuned to the needs of each client. SRP, on the other hand, calls for a partitioning of code based on different users and their needs. The modules thus partitioned by SRP may be consumed by different clients that each use different parts of the module. ISP ensures that these clients don't have to know about parts of the module they don't consume. A difference worth mentioning is that the consumer/client of ISP is a code module whereas the user of SRP is a person(s). Stated another way, ISP is the **principle of least knowledge** similar to the principle of least privilege in security.
	
**Examples:**

1. A system for realtors has `Property` objects that represent individual properties. The realtor is the user in this case and the `Property` class adheres to SRP because it only serves the realtor. The `Property` class implements an `IDetail` interface that provides detailed property information for the property detail screen. It also implements an `ISummary` interface for a master screen that lists properties matching search criteria along with property summary info. The summary screen doesn't need to know about property details, and therefore shouldn't be forced to depend on a fat interface that has both detail and summary info. 

2. CRUD(Create-Read-Update-Delete) Repository functionality can be split into IReader and (IReadWriter : IReader) interfaces. Clients that only need to query can use IReader. Clients needing to do both (typically no clients would only want to update) can use IReadWriter.<sup id="cite7">[7](https://jeremybytes.blogspot.com/2013/09/applying-interface-segregation.html)</sup>

3. A CQRS<sup id="cite9">[9](https://martinfowler.com/bliki/CQRS.html)</sup> (Command Query Request Segregation) repository splits reads and writes into IQuery and ICommand for query-only clients and clients that need to update the repository. 

### The polyad vs the monad:

This is an example from Robert Martin's ISP article<sup>[1](https://drive.google.com/file/d/0BwhCYaYDn8EgOTViYjJhYzMtMzYxMC00MzFjLWJjMzYtOGJiMDc5N2JkYmJi/view)</sup>. 
If a function `foo` depends on interfaces `IA` and `IB` that are part of a container interface `IG`, then which of these function signatures is preferable:

```csharp
// polyadic form
void foo(IA a, IB b) 
{
    // call methods on a and b	
}

// monadic form
void foo(IG g) 
{
    a = g.A;
    b = g.B;

    // call methods on a and b
}
```

According to Martin, the polyadic form is preferable to the monadic form because it ensures that function `foo`'s dependencies are clearly articulated, and it isn't saddled with dependencies it doesn't use. In the monadic example, if unrelated methods are later added to interface `IG`, then the module containing function `foo` would need to be recompiled. The less function `foo` knows about its dependencies, the more decoupled it is from them. Less cohesion between parts results in less friction from change and a more plastic system that is resilient and even welcoming to change.

Finally, some food for thought. What do you think about apps that are built on some least-common denominator framework (like Electron or even the World Wide Web)? Do they violate ISP by providing a general-purpose client that works on all types of devices using a single codebase? Do the clients (the apps themselves) have dependencies that they don't really need? Would having client-specific apps (native apps built for each client) result in a more decoupled and change-resilient system? Or do the benefits of the least common denominator framework outweigh the costs?

### Takeaways
- Interface segregation principle aims to reduce module's dependencies on each other.
- It does this by mandating that interfaces tailored to each client be used rather than general-purpose ones that try to serve all clients.
- ISP aims to provide different views of a module partitioned by SRP to different clients (that may be application classes, screens or web pages).
- ISP provides guidance on how to pass dependencies into modules, favoring enumeration of each dependency (polyadic form) rather than passing in a container that houses all the dependencies (monadic form).  

### References

1. <a href="#cite1">^</a> [The Interface Segregation Principle by Robert Martin](https://drive.google.com/file/d/0BwhCYaYDn8EgOTViYjJhYzMtMzYxMC00MzFjLWJjMzYtOGJiMDc5N2JkYmJi/view)
2. [Wikipedia](https://en.wikipedia.org/wiki/Interface_segregation_principle)
3. <a href="#cite3">^</a> [RoleInterface](https://martinfowler.com/bliki/RoleInterface.html)
4. [Interface-Segregation Principle (ISP) - Principles of Object-Oriented Class Design](https://web.archive.org/web/20100820124217/http://codebetter.com/blogs/david.hayden/archive/2005/06/15/64635.aspx)
5. [What is the distinction between SRP and ISP](https://stackoverflow.com/questions/14388358/in-solid-what-is-the-distinction-between-srp-and-isp-single-responsibility-pr)
6. https://twitter.com/unclebobmartin/status/996739060348653568
7. <a href="#cite7">^</a> [Applying the Interface Segregation Principle to a Repository](https://jeremybytes.blogspot.com/2013/09/applying-interface-segregation.html)
8. <a href="#cite8">^</a> [SOLID or Colds](https://blog.ploeh.dk/2009/09/29/SOLIDorCOLDS/)
9. <a href="#cite9">^</a> [CQRS](https://martinfowler.com/bliki/CQRS.html)