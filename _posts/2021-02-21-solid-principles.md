---
layout: post
category: blog
title: SOLID Principles
description: Overview of SOLID Principles
image: solid.jpg
---

![SOLID Principles](../../../img/solid.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@egnaro?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Rick Mason</a> on <a href="https://unsplash.com/s/photos/lego?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

New software smells great. Like a new car. It evokes images of a lush green field full of fragrant flowers, with butterflies and chirping birds. Developers usually like to build software from scratch or rewrite other people's code. They view enhancing existing software as menial labor - beneath their dignity. Like tilling an ugly brown field full of rocks, that smells of manure. Ranting about the quality of legacy code is mandatory for acceptance in the community. I've been there before.   

But legacy software is not always worthy of such disdain. In many cases, **it is an asset that has been making money for the business, a source of truth for business rules and process knowledge that has been through the fires of production usage, and will continue to be functional for many more years**. It might not need to be completely rewritten or upgraded to the latest and greatest technology platform yet. Maybe parts of it could be opportunistically redone if there is a business need or risk from using outdated technology.   

Enhancements and bug fixes would certainly be needed to support the business as it evolves. These might be more challenging to make than in [greenfield systems](https://en.wikipedia.org/wiki/Greenfield_project). [Brownfield systems](https://en.wikipedia.org/wiki/Brownfield_(software_development)) have usually accreted organically over time. Different developers have over the years brought their individual philosophies to bear on the codebase with the code itself being the only communication medium between them. Documentation is often scarce or non-existent. This is what makes system wide changes particularly challenging to make. Modifying code without breaking existing code can feel like a game of [Jenga](https://en.wikipedia.org/wiki/Jenga) at times. It is sobering to remember that **brownfield systems started life as greenfield systems**.      

According to [Bob Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) (aka Uncle Bob), **a system's design can rot over time if not tended to. This rot manifests in the form of various _code smells_:**
- *Rigidity:* Simple changes have cascading changes in dependent modules. 
- *Fragility:* Software breaks in many places when changed with breakages in areas unrelated to the change  
- *Immobility:* Code is duplicated in many places; hard to reuse code between different parts of the system  
- *Reusable code inextricably mixed with non-reusable code:* Too much work and risk to refactor 
- *Viscosity:* Harder to adhere to design than to hack system, long compile times, long running unit tests	  

This is where knowledge of SOLID principles, refactoring and [unit testing](/blog/unit-testing-1) can be invaluable. The SOLID principles are a set of Object Oriented design principles proposed by Uncle Bob to address the _code smells_ mentioned above. The key idea tying the principles together is to avoid coupling classes to each other. That is, eliminating dependencies between classes and keeping them as lean, single purpose, loosely coupled units with few or no close friends, that interact with other classes through well defined contracts. That can be developed, modified and tested in isolation and are fungible - i.e. can be snapped in and out easily like lego pieces as long as the shapes fit. **This allows us to create software architectures that are [antifragile](https://en.wikipedia.org/wiki/Antifragile), that is, not only resilient to stressors, but actually benefiting from them.** Not all the principles are equally relevant or applicable to all environments. For example, the Liskov and Open-Closed principles may not really be relevant to functional programming. On the other hand, Single responsibility and dependency inversion could certainly be.     

SOLID itself is one of those manufactured acronyms that the programming landscape is littered with. Baffling to the uninitiated, yet valuable as mnemonic devices. The principles enumerated are:  

**S**[ingle Responsibility Principle](/blog/single-responsibility-principle)  
- A class should only have a single responsibility  
- e.g. a class that processes a credit card payment should not also update the database   

**O**[pen-Closed Principle](/blog/open-closed-principle)
- Classes should be open to extension but closed for modification  
- proposed by [Bertrand Meyer](https://en.wikipedia.org/wiki/Bertrand_Meyer), it originally referred to OO inheritance
- During the 1990s, this principle was repurposed to refer to polymorphism, where interfaces can be extended by multiple implementations, but the interface specification itself is closed for modification.  

**L**[iskov Substitution Principle](/blog/liskov-substitution-principle)
- proposed by [Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov)
- It should be possible to substitute objects with instances of their subtypes (polymorphism)  
- This is key to not tying classes to concrete dependencies, but rather to abstractions  

**I**[nterface Segregation Principle](/blog/interface-segregation-principle) 
- Favor fine grained interfaces tailored to needs of different clients than one thick general purpose interface  
- Single responsibility applied to interfaces  
- Principle of least privilege - clients don't need to know about functionality they don't use, so that they don't need to adapt to changes in that functionality    

**D**[ependency Inversion Principle](/blog/dependency-inversion-principle)
- Classes should depend only on other abstract interfaces, not other concrete classes  
- A class' dependencies should be injected into the class (say, via constructor arguments) vs being instantiated within the class
- Liskov Substitution makes Dependency Inversion possible   

In future posts, we will examine each principle in more detail with examples.  

### Takeaways:
- SOLID principles are a set of Object Oriented design principles proposed by Bob Martin.
- They are a guide to building loosely coupled software architectures that are [antifragile](https://en.wikipedia.org/wiki/Antifragile).
- The principles can be applied to minimize or eliminate many design smells that a system accumulates over time.
- SOLID is an acronym that stands for Single Responsibility Principle, Open Closed Principle, Liskov Substitution Principle, Interface Segregation Principle and Dependency Inversion Principle.