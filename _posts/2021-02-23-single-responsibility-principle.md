---
layout: post
category: blog
title: Single Responsibility Principle
description: Single Responsibility Principle (SOLID Principles)
image: srp.jpg
---

![Naming](../../../img/srp.jpg)
<span class="credit">Photo by <a href="https://unsplash.com/@rossf?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Ross Findon</a> on <a href="https://unsplash.com/s/photos/change?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

> When you write a software module, you want to make sure that when changes are requested, those changes can only **originate from a single person**, or rather, a single tightly coupled group of people representing a single narrowly defined business function. You want to **isolate your modules from the complexities of the organization as a whole**, and design your systems such that each module is responsible (responds to) the needs of just that one business function.
<br>- Bob Martin

The Single Responsibility Principle (SRP) states that a class (or module) must have only **one reason to change**.  As Bob Martin clarifies, the **responsibility/reason to change** implied here is not related to a business object, but rather **a person**. A user for whom the code is written. 

For example, given a banking application with an `Account` class that represents a bank account, consider the following use cases:
- Finance wants a formatting change to the monthly account summary report
- Marketing wants to add a feature whereby deposits are rounded to the nearest dollar
- The DBA wants to change the Account table schema to capture a modified by user column

These changes are all related to the `Account` class. We could put all the code in the `Account` class. 

```
Account
+ GenerateMonthlySummaryReport()
+ RoundToNearestDollar(double amount)
+ Save()
```

Finance wants to see only rounded amounts in their report. The developer making this change inadvertently rounds transaction amounts in the Account class itself. Now, the account balances are incorrect. When business finds out, their confidence in the ability of the developer and the stability of the system is shattered. A perception takes root in their minds that the system is inherently fragile. 

To make matters worse for the developer, since Marketing also wanted something similar, it isn't clear whether the error is due to the finance change or the marketing change. 

Or whether it is due to the Save method being modified to conform to the DBA's new schema.

In short, its a dumpster fire. The root cause is that concerns were lumped together instead of being isolated. Changing one part of the system caused a break in another part, which isn't confidence-inspiring.

So how might we solve this problem? Through *separation of concerns* (a term introduced by Dijkstra). By separating each responsibility into its own class. There are many ways of doing this. One way might be to create the following classes:
```
Account
+ RoundToNearestDollar(double amount)

MonthlyAccountSummaryReport
+ GenerateReport()

AccountData
+ Save
```

This has the added benefit of making the code easier to write, since all the changes requested by a person are clustered together in the same place, and we don't have to check out and modify multiple modules. It makes the code easier to test as well for the same reason. Most importantly, there is no possibility of changes made for one user interfering with changes for another.

[James Ellis-Jones](https://hackernoon.com/you-dont-understand-the-single-responsibility-principle-abfdd005b137) makes a great point about SRP not implying that code should always be separated into classes or modules based on a flowchart, but rather based on what things change together. Thus, there might be times where code that is usually kept separate belongs in the same class - he cites the example of the [Active Record pattern](https://www.martinfowler.com/eaaCatalog/activeRecord.html) where the model and data access are combined into a single class. This pattern might make sense if there is no DBA and the developers are responsible for code and database.     

To sum up, in the words of Bob Martin:
> As you think about this principle, remember that the reasons for change are **people**. It is **people** who request changes. And you don’t want to confuse those people, or yourself, by mixing together the code that many different people care about for different reasons.


### Takeaways
- Decompose the system into classes or modules based on **what parts are likely to change together**.
- Every class should have a **who** - the user it is written for. If a class has more than one **who**, it is probably time for refactoring.
- SRP ensures **separation of concerns**, which allows a class to "focus its attention upon some aspect" of the business model instead of being a jack-of-all-trades.

### References
- [The Single Responsibility Principle](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)
- [Bob Martin SRP talk](https://www.youtube.com/watch?v=Gt0M_OHKhQE)
- [You don't understand the Single Responsibility Principle](https://hackernoon.com/you-dont-understand-the-single-responsibility-principle-abfdd005b137)
- [On the role of scientific thought](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD04xx/EWD447.html)