---
layout: default
title: The Goal
description: Applications of theory of constraints to software
---
### {{ page.title }}

![The goal](../../../img/the-goal-estee-janssens.jpg)

<span class="credit">Photo by <a href="https://unsplash.com/@esteejanssens?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Estée Janssens</a> on <a href="https://unsplash.com/s/photos/the-goal?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

Finished reading [The Goal](https://www.amazon.com/Goal-Process-Ongoing-Improvement/dp/0884271951/ref=sr_1_1?crid=DAFEMRO5ALH0&dchild=1&keywords=the+goal&qid=1606981267&sprefix=the+goal%2Caps%2C216&sr=8-1) by [Eliahu Goldratt](https://en.wikipedia.org/wiki/Eliyahu_M._Goldratt), which is a management book that ostensibly talks about improvements in a manufacturing plant. However, the principles are equally applicable to software engineering.

### Takeaways
- Clearly identify the goal (of an individual or organization). For example, **the goal of a business is to make money**. The goal of a software division of a business is to help the business in its goal of making money.
- The same goal can be articulated using different measurements. For example, finance may use ROI, net profit and cash flow to state the goal while engineering might use throughput, inventory and operational expense.
- Do more of activities that take you **towards the goal** and less of activities that take you **away from the goal**. For example, rewriting a working application to use more modern technologies for maintainability will run counter to the goal if the cost of the rewrite exceeds the maintenance costs. Maybe we also need to distinguish between short-term and long-term goals. In the longer term, using outdated or unproductive technologies may exact a higher cost than the cost of a rewrite.   
- Key variables that affect the goal are **throughput** (rate of delivering finished products to the end customer), **inventory** (work in progress, finished products not yet sold) and **operational expense** (materials, labor, equipment, facilities, carrying, financing cost). This translates readily to software. Throughput is the features or applications delivered to the customer, inventory is projects in flight, operational expense is facilities, salary, hardware, software, hosting costs.
- The goal restated: **Increase throughput while decreasing inventory and operational expense.**
- Identify and augment **bottlenecks** (through backlogs) and let their capacity guide system flow to avoid backlogs from clogging the system and increasing inventory and operational expense. For example, if a team has more developers than QA, then QA can be a bottleneck. Rather than keep increasing QA's backlog, overall throughput can be increased by redirecting resources to clear the QA backlog or by using automation to increase efficiencies.
- Let flow be driven by demand to avoid increasing inventory and carrying costs/operational expenses.
- Have **reserve capacity** to meet demand surges and account for statistical fluctuations without overwhelming the system and creating cascading backlogs.
- **Optimize globally** rather than locally. For example, 
    - In a multi-team project, bottlenecks can arise in critical paths. Rather than optimizing each team's throughput individually, resources can be redirected to clear the bottlenecks and thereby improve overall project throughput. 
    - If your personal financial goal is to increase net worth, then paying off a mortgage quicker when you could get a higher rate of return elsewhere on that extra payment might locally optimize your debt burden, but at the expense of reducing your net worth.
- Have a system for **continuous improvement** (Japanese [*kaizen*](https://en.wikipedia.org/wiki/Kaizen)).
