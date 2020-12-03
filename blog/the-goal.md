---
layout: default
title: The Goal
description: Applications of theory of constraints to software
---
### {{ page.title }}

![The goal](../../../img/the-goal-estee-janssens.jpg)

<span class="credit">Photo by <a href="https://unsplash.com/@esteejanssens?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Estée Janssens</a> on <a href="https://unsplash.com/s/photos/the-goal?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

Finished reading [The Goal](https://www.amazon.com/Goal-Process-Ongoing-Improvement/dp/0884271951/ref=sr_1_1?crid=DAFEMRO5ALH0&dchild=1&keywords=the+goal&qid=1606981267&sprefix=the+goal%2Caps%2C216&sr=8-1) by [Eliahu Goldratt](https://en.wikipedia.org/wiki/Eliyahu_M._Goldratt) and started to think about its applications to software engineering.

### Takeaways
- Clearly identify the goal (of an individual or organization). For example, **the goal of a business is to make money**. The goal of a software division of a business is to help the business in its goal of making money.
- The same goal can be articulated using different measurements. For example, finance may use ROI, net profit and cash flow to state the goal while engineering might use throughput, inventory and operational expense.
- Do more of activities that take you **towards the goal** and less of activities that take you **away from the goal**. For example, rewriting a working application to use more modern technologies for maintainability will run counter to the goal if the cost of the rewrite exceeds the maintenance costs. Maybe we also need to distinguish between short-term and long-term goals. In the longer term, using outdated technologies may exact a higher cost than the cost of a rewrite.   
- Key variables that affect the goal are **throughput** (rate of delivering finished products to the end customer), **inventory** (work in progress and products not yet sold) and **operational expense** (materials, labor, equipment, facilities, carrying, financing cost). This translates readily to software. Throughput is the features or applications delivered to the customer, inventory is the projects in progress and finished products not yet sold to a customer, operational expense is the facilities, salary, hardware, software, hosting costs.
- The goal restated: **Increase throughput while decreasing inventory and operational expense.**
- Identify **bottlenecks** (aka constraints - look at backlogs) and let their capacity guide system flow to avoid backlogs from clogging the system and increasing inventory and operational expense
- Let flow be driven by demand to avoid increasing inventory and carrying costs and thereby operational expenses
- Have **reserve capacity** to meet demand surges and account for statistical fluctuations without overwhelming the system and creating cascading backlogs
- **Optimize globally** rather than locally. For example, 
    - If the goal of a software team is to deliver a certain feature set to the user in a given time period, and the team is short on QA resources, the team's overall throughput would benefit more from developers pitching in to clear the QA backlog or writing automated tests rather than continuing to write code and increase the size of the backlog.  
    - If your personal financial goal is to increase net worth, then paying off a mortgage quicker when you could get a higher rate of return elsewhere on that extra payment might locally optimize your debt burden, but at the expense of reducing your net worth.
- Have a system for **continuous improvement** (Japanese [*kaizen*](https://en.wikipedia.org/wiki/Kaizen)).
- Abstracting from the above, the core problem of management can be stated as:
    - identify what to change
    - what to change to
    - how to effect the change

### What is the primary goal of a software engineer? (Work in progress)
- To make money (too generic - this is the goal of all professions)
- To stay relevant (secondary goal)
- To create a body of work (secondary goal)
- To solve interesting problems (this is a byproduct)
- To design the simplest system that satisfies the given constraints (features, cost, performance)
- To architect systems that are correct, reliable and maintainable
- Continuous improvement
- **To free the mind for creative thought** by automating repeatable tasks
- **To create tools for the mind**
- **To unlock human potential**
- **To help us lead richer lives**
- To access and gain insights from the entirety of personal and human knowledge
- To help businesses make money by improving every aspect of doing business
- "That the powerful play goes on, and you may contribute a verse." to channel Robin Williams (Dead Poets Society) and the [Apple commercial](https://www.youtube.com/watch?v=Ep2_0WHogRQ) 

### What are the blockers to this?
- Lack of knowledge
- Lack of communication
- Lack of experience
- Lack of proper tools
- Lack of interesting problems
- Lack of clear requirements
- Lack of quality control
- Lack of finances
- Lack of health
- Lack of motivation
- Lack of vision
- Lack of imagination
- Lack of nerve
