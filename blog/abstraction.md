---
layout: default
title: Abstraction
description: Concept of abstraction in software design
---
### {{ page.title }}

![Abstraction](../../../img/abstraction-ricardo-gomez.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@ripato?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Ricardo Gomez Angel</a> on <a href="https://unsplash.com/s/photos/abstract?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

Complex systems (natural or man-made) can be understood as a set of concentric abstractions.

An abstraction is a label given to a subsystem that hides internal details unnecessary to understanding its external interactions. For example, to understand the motion of a projectile under gravity, it is not necessary to know about its chemical composition or molecular structure. To understand the motion of a car, one does not need to know how an internal combustion engine works. A pilot does not need to know about the physics of flight in order to fly a plane.

A computer program or computational process is likewise composed of a ladder of abstractions. Each rung of the ladder builds on the rungs below it. 

A function or a procedure is a powerful abstraction that lets us refer to a complex sequence of operations by giving it a name. Abstractions let us think in terms of what rather than how, which is internal implementation detail. They also let us identify patterns that could be reused, thereby simplifying a system's design. 

For example, the function `sqrt-iter` below is used to calculate the square root of a number using [Newton's method](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-10.html#%_sec_1.1.7):

```clojure
(define (sqrt-iter guess x)
  (if (good-enough? guess x)
      guess
      (sqrt-iter (improve guess x)
                 x)))
``` 

From [Structure and Interpretation of Computer Programs](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/book-Z-H-10.html#%_sec_1.1.8):
- "Observe that the problem of computing square roots breaks up naturally into a number of **subproblems**:
how to tell whether a guess is good enough, how to improve a guess, and so on. Each of these tasks is
accomplished by a separate procedure. The entire sqrt program can be viewed as a **cluster of procedures
(shown in figure [1.2](https://mitpress.mit.edu/sites/default/files/sicp/full-text/book/ch1-Z-G-6.gif)) that mirrors the decomposition of the problem into subproblems.**

- "The importance of this decomposition strategy is not simply that one is dividing the program into parts. After all, we could take any large program and divide it into parts -- the first ten lines, the next ten lines, the next ten lines, and so on. Rather, it is crucial that **each procedure accomplishes an identifiable task that can be used as a module in defining other procedures**. For example, when we define the `good-enough?` procedure in terms of `square`, we are able to regard the `square` procedure as a '**black box**'. We are not at that moment concerned with how the procedure computes its result, only with the fact that it computes the square. **The details of how the square is computed can be suppressed, to be considered at a later time.**"

- "Indeed, as far as the good-enough? procedure is concerned, `square` **is not quite a procedure but rather an abstraction of a procedure, a so-called procedural abstraction. At this level of abstraction, any procedure that computes the square is equally good.**"

Abstraction is a simple yet profound concept that is overlooked by even senior engineers including yours truly. It is not uncommon to come across code running into several pages that looks somewhat like this:

```javascript
function purchase(order) {
    // check if order is valid
    .... step 1
    .... step 2
    .... step 3
    // charge credit card
    .... step 1
    .... step 2
    .... step 3
    // confirm purchase
    .... step 1
    .... step 2
    .... step 3
    // email receipt
    .... step 1
    .... step 2
    .... step 3
}
``` 

instead of this:

```javascript
function purchase(order) {
  if (!order.IsValid())
    return invalid_order;
  if (!charge(order)) {
    return charge_failed;
  }
  if (!confirmPurchase(order)) {
    return confirm_failed;
  }
  email_receipt(order);
}
```

In the former, it is easy to get lost in the weeds of the details of each step and not grasp the gist of what the function is trying to accomplish. In the latter, it is evident what the function's high-level purpose is because the details of each operation have been abstracted away into functions that convey the sequence of operations without clouding the waters with implementation details. 

Another example of this is an application where there is no clear separation of presentation, business and data access logic, making it hard to independently test and swap out layers (e.g. replace database calls with mock calls for unit tests).

To understand and precisely communicate abstractions cross-functionally and across organizations, a consistent vocabulary is required. This is referred to as an [Ubiquitous Language](https://martinfowler.com/bliki/UbiquitousLanguage.html) (or just a plain glossary to keep it simple). 
This makes it easy to understand different interconnected systems without having to mentally translate between different terms that refer to the same thing. 

Most architectural patterns and principles have abstraction at their core.

#### Takeaways
- Think about systems and processes in terms of abstractions
- Abstractions help us think in terms of what (breadth) instead of how (depth)
- Abstractions help us understand interconnections between parts of a system
- Abstractions help us identify reusable patterns
- Spend time on naming things
- Use consistent naming across the organization