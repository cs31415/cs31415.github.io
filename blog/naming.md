---
layout: default
title: Naming
description: Naming
image: naming.jpg
---
### {{ page.title }}

![Naming](../../../img/naming.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@waldemarbrandt67w?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Waldemar Brandt</a> on <a href="https://unsplash.com/s/photos/name?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

>There are only two hard things in Computer Science: cache invalidation and naming things.<br/>
>-- _Phil Karlton_

Naming is one of those unglamorous things about programming that is mostly in our peripheral vision during the learning phase. Our attention is focused on more exciting things like algorithms and graphics. This apathy usually spills over into the professional phase. It takes some catastrophe, some inexplicable behavior that gives us sleepless nights, traceable to some obscure method whose name isn't remotely indicative of things it is actually doing, to bring naming into sharp focus as a topic worthy of attention.

Naming and [abstraction](/blog/abstraction) are closely intertwined. Names are how we identify abstractions - those black boxes, those legos that we can click together without needing to know how they were manufactured. A good abstraction represents a single concept and a good name helps us identify that concept instantly with no further elaboration required. Good abstractions aided by good names minimize cognitive overhead and help us read code like prose says 
[Grady Booch](https://en.wikipedia.org/wiki/Grady_Booch):
> Naming things properly - so that they reflect their semantics - is often treated lightly by most developers, yet is important in **capturing the essence of the abstractions** we are describing. Software should be written as carefully as English prose, with consideration given to the reader as well as to the computer. - [Object Oriented Design With Applications](https://www.amazon.com/Object-Oriented-Analysis-Design-Applications-2nd/dp/0805353402/ref=sr_1_2?dchild=1&keywords=object+oriented+design+booch&qid=1612855563&sr=8-2)

According to [The Pragmatic Programmer](https://www.amazon.com/Pragmatic-Programmer-Journeyman-Master/dp/020161622X), avoiding misleading names is even more important than avoiding inane names: 
> Variable names of course, should be well chosen and meaningful. Even worse than meaningless names are __misleading__ names. Have you ever had someone explain inconsistencies in legacy code such as, "The routine called `getData` really writes data to disk"? The human brain will repeatedly foul this up - its' called the __Stroop Effect__... **Names are deeply meaningful to your brain, and misleading names add chaos to your code.**"

Naming system artifacts well is also a form of documentation. It lets us hone in on an area of code quickly, be it to understand and extend it or to fix an urgent problem. Using the same consistent terminology in the user interface and back end code is a huge time saver. Too many person hours and brain cells have been lost trying to map user interface elements to a backing object. 

Names are not just a programming language requirement that we must grudgingly assign and be done with. They also form a compressed language that users and developers use to communicate elaborate concepts with each other. The more precise this language, the less scope for ambiguity and misinterpretation of requirements. [Eric Evans](https://www.amazon.com/Eric-Evans/e/B001KDCO2I%3Fref=dbs_a_mng_rwt_scns_share) recommends creating a glossary of terms at the outset, which he calls [Ubiquitous Language](https://martinfowler.com/bliki/UbiquitousLanguage.html), so that users and developers can be assured they are talking about the same things. 

Names can also help identify code smells. For example, a method named `FetchAndLoadData` smells of a violation of [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle). A method or class shouldn't have multiple responsibilities. Of course, not all code smells are likely to be so conscientiously labeled. It is more likely that a method named `FetchData` will also load it into the database. We need to watch out for both of these smells and opportunistically refactor whenever possible. In the first instance, by spinning off a `LoadData` method and renaming the original method to `FetchData`, and in the second instance by spinning off a `LoadData` method. 

[Design Patterns](https://en.wikipedia.org/wiki/Design_Patterns) are another kind of ubiquitous language for developers to communicate common solution archetypes with each other. Naming classes using the pattern vocabulary (factory, adapter, singleton, visitor, etc.) helps efficiently and clearly communicate their purpose, object structure and relationships using a single word instead of resorting to diagrams and lengthy explanations. It also enables reuse of solutions that might not have been evident but for the shared vocabulary.

Conventions I usually follow in C# code are:
- private member variables (camel case starting with lower case and prefixed by underscore)
<br>e.g. `private int _count;`
- public properties (camel case starting with upper case)
<br>e.g. `public int Length { get; set; }`
- classes (camel case starting with upper case)
<br>e.g. `OrderManager`
- methods (camel case starting with upper case)
<br>e.g. `ConfirmOrder`
- interfaces (starting with capital I)
<br>e.g. `IOrderManager`
- local variables & method arguments (camel case starting with lower case)
<br>e.g. `orderManager`

Looking at .NET framework code and following the same conventions shouldn't be a bad idea. Neither is formalizing these kind of conventions in a coding guidelines document for a project, and even better, automatically enforce them using linters. Future maintainers, which might include your future self, will be eternally grateful.

Finally, it is OK to take a little time upfront naming things. It might seem that this time is better spent writing code. But given that language shapes our thought, writing code using the wrong abstractions is a recipe for problems down the line that will be harder to fix without requiring significant design changes. Naming doesn't come easily to even someone as accomplished as [DHH](https://dhh.dk/). His [inner dialogue](https://m.signalvnoise.com/hunting-for-great-names-in-programming/), as he obsesses over a method name is both fascinating and insightful to observe.    

### Takeaways:
- Names identify [abstractions](/blog/abstraction)
- Take time to name things, iterating multiple times if required
- Respect [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single-responsibility_principle) when naming things
- Poor names are indicative of ill-defined abstractions
- Names are a form of documentation that allow us to quickly identify an affected area of code
- Names are a communication language for users and developers
- Maintain a glossary of terms for each project that developers and users adhere to (aka [ubiquitous language](https://martinfowler.com/bliki/UbiquitousLanguage.html))
- Consistently follow naming conventions across a codebase, formalizing them in a coding guidelines document or linter rules

### References:
- [The importance of naming in programming](https://carlalexander.ca/importance-naming-programming/)
- [Is naming things really that hard?](https://wade.be/development/2017/03/03/naming-things.html)
- [Why naming things is hard](https://hilton.org.uk/blog/why-naming-things-is-hard)
- [Ubiquitous Language](http://www.jamesshore.com/v2/books/aoad1/ubiquitous_language)
- [Hunting for great names in programming](https://m.signalvnoise.com/hunting-for-great-names-in-programming/)
- [Clean Code: Naming Conventions](https://thecoderoad.blog/2020/03/29/clean-code-naming-conventions/#:~:text=According%20to%20Uncle%20Bob%20in,%E2%80%9Cintention%2Drevealing%E2%80%9D%20names)

