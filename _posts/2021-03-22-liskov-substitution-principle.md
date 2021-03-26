---
layout: post
category: blog
title: Liskov Substitution Principle
permalink: /blog/liskov-substitution-principle
description: Liskov Substitution Principle (SOLID Principles)
image: lsp.jpg
---

![Liskov Substitution Principle](../../../img/lsp.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@kaffeemeister?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Kaffee Meister</a> on <a href="/s/photos/oat-milk?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></span>

The Liskov Substitution Principle(LSP), also known as *behavioral subtyping*, was formulated by [Turing Award](https://en.wikipedia.org/wiki/Turing_Award) winner [Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov), an accomplished computer scientist who also proposed the notion of [Abstract Data Types](https://en.wikipedia.org/wiki/Abstract_data_type) among other things.

In the words of Liskov herself:
> Objects of a subtype ought to behave the same as those of the supertype as far as anyone or any program using supertype objects can tell.<sup>[3]</sup>

In other words, it should be possible to drop in a subtype wherever a supertype is expected and the code should not just compile, but also function as expected at runtime. 

> I think I can safely say that of the SOLID principles, the Liskov principle is by far the least understood and applied.<sup>[1]</sup> - Dino Esposito

This is a subtle principle that might seem trivial at first sight. Isn't this just how polymorphism is supposed to work? Well, compilers can only validate polymorphism syntactically. They have no way of doing a semantic (meaning) check. It is this gap that the LSP seeks to address. Subtypes can't just override supertype methods willy nilly. They have to ensure that clients aren't confused when a subtype instance is passed in where a supertype is expected.

Compilers cannot flag violations of the Liskov principle. The only way we can identify a violation is by validating the subtype's behavior against the expected behavior of the supertype. The expected behavior of supertypes then, must be documented in the code, and implementors of subtypes must adhere to it, otherwise unexpected and hard-to-detect runtime errors might result. LSP violations usually indicate a problem with the inheritance hierarchy. Perhaps the subtypes don't truly satisfy the "is a" relation.  

LSP implies the following corollaries<sup>[4]</sup>:
-   [Preconditions](https://en.wikipedia.org/wiki/Precondition) (conditions true prior to method execution) cannot be strengthened in the subtype.
In other words, the subtype method’s domain (set of possible inputs) must be a **superset** of supertype method. 
-   [Postconditions](https://en.wikipedia.org/wiki/Postcondition) (conditions true post method execution) cannot be weakened in the subtype.
The subtype method's range (set of possible outputs) must be a **subset** of supertype method. 
-   [Invariants](https://en.wikipedia.org/wiki/Invariant_(computer_science)) (data that remains unchanged by the method) must be preserved in the subtype.

Intuitively, this makes sense. If the subtype can handle all the inputs that the supertype can (and some more), and output values that don't exceed the range of values output by the supertype, and values that the supertype doesn't change remain unchanged by the subtype, then clients that work with a supertype instance can be expected to work with a subtype instance as well.  

The LSP can be better understood by looking at some examples of violations:

#### Missing implementations of supertype methods:

Subtype methods with missing implementations are a violation of the LSP. If the client expects the supertype to have a certain method and the subtype fails to implement it, then the subtype fails the "is a" supertype test. In the example below, `ITicketingAgent` represents an interface to a movie theater. Theaters can have auditoriums with reserved seating or general seating. 

```
ITicketingAgent
+ bool IsShowSoldOut(IShow show)
+ string[] GetAvailableSeats(IReservedShow show)
+ ReserveSeats(IReservedShow show, string[] seatIds)
+ ReserveSeats(IShow show, int numSeats)
```

Bilbo's theaters don't support reserved seating. Hence the `GetAvailableSeats` and `ReserveSeats` method taking a list of seat ids is not implemented.

```
BilbosTicketingAgent : ITicketingAgent
+ bool IsShowSoldOut(IShow show)
+ string[] GetAvailableSeats(IReservedShow show) <= not impl
+ ReserveSeats(IReservedShow show, string[] seatIds)  <= not impl
+ ReserveSeats(IShow show, int numSeats)
```

Frodo's theaters only have auditoriums with reserved seating. Hence, the methods for general seating auditoriums aren't implemented. Both of these are violations of the LSP. It indicates our original ticketing agent abstraction may be too broad.

```
FrodosTicketingAgent : ITicketingAgent
+ bool IsShowSoldOut(IShow show)  <= not impl
+ string[] GetAvailableSeats(IReservedShow show)
+ ReserveSeats(IReservedShow show, string[] seatIds)
+ ReserveSeats(IShow show, int numSeats)  <= not impl
```

#### Pre-conditions violation:

> The key point is that a derived class can’t just add preconditions. In doing so, it will restrict the range of possible values being accepted for a method, possibly creating runtime failures. - Dino Esposito <sup>[1]</sup>

In this example, we have `SimpleZip` and `AdvancedZip` classes that compress a given string. They are both subtypes of the `IZip` interface. `AdvancedZip` adds the precondition that input should not be null, where the base interface specifies no such expectation. This is a violation of the LSP precondition rule. The thing to note here is that LSP depends on what expected behavior is specified in the base class. If the interface in this example had specified that null input would raise exception, then `SimpleZip`, not `AdvancedZip` would violate LSP.
```
interface IZip
{
    // returns null if input is null else returns compressed string
    string Zip(string input);
}

class SimpleZip : IZip
{
    public string Zip(string input)
    {
        if (input == null)
            return null;
        // compress spaces
        return Utils.CompressSpaces(input);
    }
}

class AdvancedZip : IZip
{
    public string Zip(string input)
    {
        if (input == null)
            throw new ArgumentException();
        return Utils.CompressRepeatingBlocks(Utils.CompressSpaces(input));
    }
}
```

#### Post-conditions violation:

The canonical example flogged to death by every other article discussing LSP is revisited here. `Rectangle` can be stretched by a multiplicative factor. `Square` rather unwisely piggybacks on `Rectangle`, and boxes the `Height` and `Width` to be always equal. `HorizontalStretch` is expected to only increase the `Width` and keep `Height` invariant. However, since `Height` and `Weight` move in concert for `Square`, `Height` is no longer invariant to `HorizontalStretch`. If the client was rendering the shapes and had sized the canvas expecting the `Height` to remain unchanged, part of the `Square` would now be clipped. If we overrode `HorizontalStretch` for `Square` to keep `Height` invariant, it would no longer remain a `Square`. Damned if you do, damned if you don't. A good sign that the hierarchy isn't sound.

```
class Rectangle
{
    public virtual int Height { get; set; }
    public virtual int Width { get; set; }

    // postcondition: width = factor * width
    // invariant: height
    public virtual void HorizontalStretch(int factor)
    {
        Width *= factor;
    }

    public virtual void Print()
    {
        Console.WriteLine($"Height = {Height}, Width = {Width}");
    }
}

class Square : Rectangle
{
    private int _height;
    public override int Height
    {
        get => _height;
        set => _height = _width = value;
    }

    private int _width;
    public override int Width
    {
        get => _width;
        set => _width = _height = value;
    }
}
```

All this is well and good, but who nowadays is using inheritance for code reuse? Is the LSP even relevant anymore? It very much is<sup>[14]</sup>, asserts Bob Martin, since the LSP was never about inheritance and all about subtyping. Implementing an interface is subtyping. Even in dynamic languages such as JavaScript with no formal notion of interfaces, [duck typing](https://en.wikipedia.org/wiki/Duck_typing) (where an object just has the same methods as a base object's and no formal hierarchy exists) implies an underlying interface with certain expected behavior. Subtypes violating that expected behavior pay the price in the form of runtime errors and/or code defecation in the form of if/else/switch statements with special logic for specific types. 

There is a lot more to pre/post-conditions violations than has been said here that requires a deeper dive into topics like [covariance and contravariance](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)). [Eric Lippert](https://ericlippert.com/) has written a series of [posts](https://docs.microsoft.com/en-us/archive/blogs/ericlippert/covariance-and-contravariance-in-c-part-one) on this topic that should be locatable with some Google skills. 

### Takeaways
- Liskov Substitution Principle says that subtypes should not alter expected behavior of supertypes.
- Passing a subtype in lieu of a supertype should be totally transparent with no special handling required on the part of clients.
- LSP implies that preconditions may not be tightened, postconditions not weakened, and invariants not modified by a subtyped method.
- Violations of LSP may indicate a flawed hierarchy.
- Since compilers can't flag LSP violations, expected behavior of base classes/interfaces must be well documented in the code.

### References
1. [Cutting Edge - Code Contracts: Inheritance and the Liskov Principle](https://docs.microsoft.com/en-us/archive/msdn-magazine/2011/july/msdn-magazine-cutting-edge-code-contracts-inheritance-and-the-liskov-principle)
2. [Barbara Liskov: The Liskov Substitution Principle](https://www.youtube.com/watch?v=-Z-17h3jG0A)
3. [A Behavioral Notion of Subtyping - B. Liskov, Jeannette Wing](http://web.cse.ohio-state.edu/~soundarajan.1/courses/788/lwb.pdf)
4. [Wikipedia: Liskov Substitution Principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle)
5. [SOLID Design Principles Explained: The Liskov Substitution Principle with Code Examples](https://stackify.com/solid-design-liskov-substitution-principle/)
6. [SOLID violations in the wild: The Liskov Substitution Principle](https://www.devonblog.com/software-development/solid-violations-wild-liskov-substitution-principle/)
7. [Is this a violation of the LSP?](https://softwareengineering.stackexchange.com/questions/170138/is-this-a-violation-of-the-liskov-substitution-principle/170142)
8. [Example of LSP Violation using vehicles](https://stackoverflow.com/questions/20861107/can-anyone-provide-an-example-of-the-liskov-substitution-principle-lsp-using-v)
9. [Violating LSP](https://www.codeproject.com/Articles/648987/Violating-Liskov-Substitution-Principle-LSP)
10. https://stackoverflow.com/questions/35582996/liskov-substitution-principle-violations
11. https://softwareengineering.stackexchange.com/questions/187613/how-does-strengthening-of-preconditions-and-weakening-of-postconditions-violate
12. https://softwareengineering.stackexchange.com/questions/170189/how-to-verify-the-liskov-substitution-principle-in-an-inheritance-hierarchy
13. https://www.ckode.dk/programming/solid-principles-part-3-liskovs-substitution-principle/
14. [SOLID Relevance](https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html)