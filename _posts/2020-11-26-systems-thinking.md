---
layout: post
category: blog
title: Systems Thinking
description: Systems thinking in software design 
image: systems-thinking-alexander-schimmeck.jpg
---

![Circuit board](../../../img/systems-thinking-alexander-schimmeck.jpg)

<span class="credit">Photo by <a href="https://unsplash.com/@alschim?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Alexander Schimmeck</a> on <a href="https://unsplash.com/s/photos/printed-circuit-board?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>


A system is a network of __interconnected__ components/actors that accomplishes a purpose.
For example, an e-commerce site, the market, government, a company, a printed circuit board, a microprocessor are all systems. Systems can exist at different levels of [abstraction](../abstraction) (e.g hardware, software, business process, etc.)

Systems thinking refers to thinking in terms of the system as an interconnected whole rather than as a collection of disconnected parts.
This is what lets us understand how the system is affected by **feedback**, for example or how to clear bottlenecks, or how to improve global rather than local throughput. 

#### Why learn systems thinking?
- Understand how a part being worked on relates to the system as a whole
- Answer the question why a part is doing what it is doing (before understanding how)
- Avoid solving the wrong problem. The real problem might not be with a component's logic but with upstream data and fixing the problem upstream might be a better solution since it fixes it for all consumers of that data. 
- Optimize globally rather than locally. For example, if there is a QA bottleneck due to resource constraints, developers cranking out more work isn't going to increase overall system throughput. Temporarily reassigning devs to clear the bottleneck will help increase system throughput.

#### Takeaways
- Draw a diagram(s)
- Sketch out the external interfaces of a component (ingress/egress points) to understand inter-relationships between parts
- Review functional specifications and integration tests to understand the goal of the system
- Seek to answer the question why a part is doing what it is doing before understanding how it does it
- Ask a business or product person to explain the business model and answer the questions:
    - How does the business makes money?
    - How does the system help the business make money? (for example, a sales funnel and a backend job to load product data both help make money - the former directly, the latter indirectly by providing the data required by the former)

#### References
1. Hanselman, Scott. [Systems Thinking As Important as Ever for New Coders](https://www.hanselman.com/blog/systems-thinking-as-important-as-ever-for-new-coders)
2. Kim, Daniel. [Introduction to Systems Thinking](https://thesystemsthinker.com/introduction-to-systems-thinking/)
3. Goldratt, Eliahu. [The Goal](https://www.amazon.com/Goal-Process-Ongoing-Improvement/dp/0884271951)
