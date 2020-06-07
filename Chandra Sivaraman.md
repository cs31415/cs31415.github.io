# Chandra Sivaraman
<sup>chandra.sivaraman@loscode.com</sup>

## SUMMARY
- Backend developer/application architect, self-starter, team player, patterns/practice-orientation, APIs/automation/serverless/tools/frameworks, C#/.NET/SQL/nodeJS/AWS, clean code/naming enthusiast
- Rearchitected Fandango mobile ticketing backend to use MVC and REST microservices, detangle concerns and dependencies, be more testable; this architecture supports 100% ticketing transaction load, enables quicker time to market, and accomodates steep traffic spikes during marquee pre-sales/release events
- Architected ticketing APIs for internal use and for Fandango partners including Google, Facebook and Amazon using Web API and nodeJS in a hybrid architecture; the APIs enabled strategic partnerships and laid groundwork for refactoring monolithic backend
- Designed low-maintenance data-driven backoffice automation jobs for sales tax calculation, data retention policy, etc. using on-premises and AWS serverless architecture to enable compliance
- Architected P&L report automation and data reconcilation tool for hedge fund to support real-time decision-making and internal data consistency
- Architected production-grade loan-level pricing models at Bank of America to enable MBS pool formation/ trading and free up capital to restart loan lifecycle
- Architected automated system at Bank of America to surface cross-application, hardcoded business rules through queryable interface; empowered analysts to gain deep insight into business logic without requiring source code reading skills
- Architected order processing and task management frameworks at banks and financial startups to offload backend processing from user interfaces, scale with order volume, improve resilience through throttling and retry mechanisms and enable scheduled task execution

## SKILLS
- C#, ASP.NET, MVC, Web API, .NET Core, XUnit, Moq
- JavaScript, nodeJS, express, Mocha, Chai, Sinon
- SQL server, ElasticSearch
- AWS lambda, step functions, Terraform, ELK

## PROJECTS
### Jul 2012-current Staff Engineer at Fandango, Beverly Hills, CA

* Created set of tools using SQL, C# parsers to inventory SQL calls, referenced SQL objects and associated SQL operations in a given codebase. This enabled DBAs to tighten DB permissions by application for cloud migration project. Effected huge time-savings and improved accuracy over manual scan.  
**Technologies:** C#, Roslyn code analysis API, TSql parser API
* Architected, implemented cloud batch workflow for calculating sales tax on ticketing transactions using AWS lambda/step functions. Architected workflow for cleansing theater addresses using lambda/step functions and task parallelism to reduce total run time.  
**Technologies:** C#, .NET Core, AWS lambda/step functions, Terraform
* Architected, implemented REST microservices for ticketing, returns and reserved seating functionality enabling integration with major social media platforms. Designed stateless API wrapper around legacy monolithic code. Built nodeJS API gateway layer to aggregate and expose microservices.  
**Technologies:** nodeJS, C#, .NET, Web API, Elastic Search, SQL Server, AWS, Docker, Terraform
* Architected and led development of ticketing backend rearchitecture project using MVC and Web API. Refactored and repurposed complex legacy classes and created comprehensive test suite for quality control.  
**Technologies:** C#, .NET, ASP.NET MVC, SQL Server, JQuery
* Architected and implemented table-driven automation job to cleanse PCI/PII transactional data to implement data retention policy. Table-driven architecture ensured system was low maintenance and able to accomodate future requirements with minimal effort.  
**Technologies:** C#, .NET, ASP.NET, SQL Server
* Designed REST microservice for VIP Plus loyalty program interfacing with  external loyalty service provider. Implemented HMAC security around loyalty redemptions to prevent abuse.  
**Technologies:** C#, .NET, Web API, SQL Server
* Architected and designed database schema for store credit feature. Integrated store credits into payment module of ticketing funnel. Incorporated decorator pattern to recalculate credit balance as different payment methods applied or removed.  
**Technologies:** C#, .NET, ASP.NET, SQL Server
* Architected and implemented rules engine for looking up service fees based on user-configurable arguments. Built admin interface to rules for users and integrated into ticketing purchase funnel. This feature provided the business with great flexibility around setting different service fees for different theaters, movies, ticket types, etc.  
**Technologies:** C#, .NET, ASP.NET, SQL Server
* Prototyped customer service chatbot to handle standard queries such as refunds, missing confirmation emails and order lookup. The goal being to  reduce customer service wait time and lower support cost. Used adaptive cards to prefilter LUIS service requests to lower operational cost.  
**Technologies:** nodeJS, adaptive cards, Microsoft LUIS  

### Feb 2010-Jul 2012 System Architect at Tennenbaum Capital, Santa Monica, CA
* Architected diff/merge tool to automate reconciliation of securities data  between trading, accounting and master databases. This eliminated manual labor and improved internal data consistency, thereby eliminating a class of problems.  
**Technologies:** C#, .NET, WinForms, SQL Server, LINQ
* Architected and implemented automated system to generate on-demand P&L for private funds in portfolio. Built backend job to aggregate multiple feeds and populate report data and SSRS report builder to shape data into P&L format.  
**Technologies:** SQL Server, SSRS, Excel pivot tables  

### Feb 2003-Feb 2010 VP Application Development at Bank of America, Calabasas, CA   
* Led design and implementation of fixed income models used for forming optimal loan pools for MBS trading. Translated rough Excel models into high-performance, production-quality code able to process hundreds of thousands of loans on daily basis.  
**Technologies:** C++, ATL, C#, .NET, SQL Server
* Architected and implemented a tool enabling analysts to gain a consolidated view of complex business rules by searching source code for multiple systems. Built a backend process to extract business rules into a database by parsing source code directly from source control, and a web interface to query the database.  
**Technologies:** C#, ASP.NET, SQL Server, Regular expressions

### Jul 2001-Jan 2003 Senior Engineer at Ocwen, Carlsbad, CA
* Architected and implemented an order processing framework with multi-threading and plug and play capabilities for a mortgage service portal.  
**Technologies:** C++, ATL, VB, COM, SQL Server

### Jan 2000-Jun 2001 Principal Engineer at NCommand, San Mateo, CA
* Extended architecture of task management framework for a B2B mortgage platform by adding round robin load balancing, throttling, retries,  scheduled execution, resiliency around scheduled downtime windows, etc.  
**Technologies:** C++, ATL, VB, COM, SQL Server

## EDUCATION
**1995** Bachelor of Electronics Engineering, VJTI - University of Mumbai, India
