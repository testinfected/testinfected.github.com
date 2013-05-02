---
layout: post
title: Simple Web Development In Java
category: Java
---
  
<div class="illustration">
	<img alt="Simplicity is the key to brilliance" title="http://thepresentationdesigner.co.uk/blog/portfolio-item/bruce-lee-slide-2/" src="/images/Bruce-Lee-Simplicity-is-the-key-2-brilliance.jpg"/>
</div>
  
*Teams often are slowed down by the increasing size and complexity of the frameworks they use and the code they write. But do they really need frameworks?* 

*The reason we often choose frameworks is that they feel easy to start with and seem to hide the complexity involved in solving complex problems. Often though they give headaches and are a pain to get rid of. If the solution to your problem does not require a framework, do not use one. Do It Yourself Simply (DIYS).*

Reading the following situations might evoke a feeling of déjà vu.

a. You start a new development effort by assembling 5 to 10 (or more) libraries as a first step. The cost of carrying those libraries proves to be high. It's not just installing. It's the overall cost of learning, knowledge transfer, keeping up to date, supporting. It's all the added complexity. 

b. A framework keeps getting in your way. You have to invest of lot of energy to work around that framework. Typically, it gets in the way of test-driven development. The framework feels like a prison: you resist abandoning it because you fear the cost of change will be to high. Your code is too tightly coupled to the framework. Instead of listening to the feedback from your tests, you let go of your good intentions.

c. A framework keeps surprising you with some magical behaviors. It has unintended and inappropriate behaviors. You wish you could explain what's happening, but cannot. You spend horrendous amounts of time searching user forums for an answer, but cannot find any. The source code is so hard to follow it won't help either. You browse Google in desperation...

### The Little Story

Some time last year, several discussions on the [Growing Object Oriented Software, Guided By Tests](http://www.growing-object-oriented-software.com) discussion group inspired me to take action and rewrite an existing Java web application using only simple tools and libraries. 

I was curious and challenged to see how I could actually build a transactional web application in Java in a simple way. By simple way I mean without a web framework, an Object Relational Mapping (ORM) framework or a Dependency Injection (DI) framework. 

The [original application](https://github.com/testinfected/petstore) is a sample application used mainly for academic purposes -- typically when teaching how to do Test-Driven Development from the outside-in in Java. It uses Spring as a DI framework, Spring MVC, Hibernate as the ORM framework and Maven as the build tool. Velocity views and SiteMesh templates render the HTML. For better or worse, it's a pretty standard Java stack -- similar to what a lot of my consulting clients are using for their day to day development. 

### The Good, the Bad and the Ugly

There is some good, some not so good and some ugly in the original application. 

<div class="illustration">
	<img alt="The Good, the Bad and the Ugly" src="/images/the-good-the-bad-and-the-ugly.jpg"/>
</div>

On the good side, the code is pretty clean and very well tested. It has been test-driven developed from the outside-in. End-to-end tests cover typical usage scenarios. Integration tests with the database cover both data access code, Spring transaction demarcation, Hibernate configuration and mapping, as well as database migrations. Unit tests for the views are done with offline HTML rendering and assertion against CSS3 selectors expressions with custom [Hamcrest](http://code.google.com/p/hamcrest/) matchers. The overall code coverage is around 99% when combining coverage from the unit, integration and end-to-end tests. 

There are also those areas were it could have been better. The domain model does not reify the concept of user requests, which results in some web controllers doing too much. Startup cost of the database integration tests is noticeable because they load Hibernate and a trimmed down Spring context. Assembling the application for running end-to-end tests is a bit tricky. Startup cost of the application also makes the end-to-end tests somewhat painful to run. 

Finally, there are the real pain points. The Maven build is one of them, as always. It packs more than 1100 lines of XML for a rather small application! Configuring the application for running the end-to-end tests requires overriding application configuration with test settings via system properties override in Spring. End-to-end and integration tests use static contexts to avoid loading spring and hibernate contexts multiple times. Last, but not least, the WAR file weighs in at 53 jars, for a rather small application. 

### The Challenge

For the rewrite, the challenge was to eliminate the pain points and to follow these guiding principles:

* Use only simple tools and libraries that are tailored to the problem at hand
* No framework, all libraries should have a single responsibility -- doing only one thing and doing it well
* Do It Yourself Simply (DIYS)
* No XML
* No annotations
* Minimum startup and shutdown cost -- so that tests can run fast
* Ease of assembly and deployment -- so that tests are easy to setup

The [new version](https://github.com/testinfected/simple-petstore) of the application is built on top of [Simple](http://www.simpleframework.org) for the webserver, standard JDBC, [Mustache](http://mustache.github.com) [Java flavor](https://github.com/samskivert/jmustache) for the view templates, and my 
own small command line interface [library](https://github.com/testinfected/cli). Everything else was test-driven developed. That is: 

* Object assembly, 
* Data mapping, 
* Transaction management, 
* Environment configuration, 
* Routing, 
* Serving static assets, 
* Access logs,
* Layout templating

That new version does not follow the Servlet specification. It uses a simple web server and builds as a single jar with embedded configuration. Most things that were left to frameworks in the original version -- namely transaction demarcation, objet assembly, wiring, routing, and database mapping -- are done programmatically in the rewrite.

### The Benefits of Simplicity

<div class="illustration">
	<img alt="Built for Speed" src="/images/built-for-speed.jpg"/>
</div>

One of the things I like the most about the new version is how fast it starts up. Startup time is down from around 6-7s on my machine for the Spring application to a negligible amount of time for the Simple version. That small startup cost makes a huge difference. It means the end-to-end tests run much faster. It also means re-deploying the application results in almost no service interruption. 

The application is self-contained -- it is packaged as a single jar -- and has no external configuration. Configuration is bundled with the application -- including test configuration. As a result, the end-to-end tests exercise the entire application through its main entry point, which results in greater test coverage. Self-contained configuration also makes it very easy to setup, startup and shutdown the application from the end-to-end tests. That means the application can be easily installed as a daemon as any other Unix tool.

Tests are blazingly fast, and that too makes a huge difference. End-to-end tests run 50% faster. They are down from 18s to 12s using Firefox and only 5s when run with PhantomJS. Database integration tests run under 1s compared to 3s in the original application. Configuring the tests for running on the test database is now straightforward. The best part is that database integration tests run so fast that it is no longer necessary to maintain a static test context. There is no penalty to bootstrapping the data mapping code on every test. 

Doing simple has also tremendous benefits in terms of total cost of ownership of the application:

* The cost of understanding the tools and technologies is low, which makes getting new team members up-to-speed much easier
* The increased startup speed has lots of benefits on testing and solves re-deployment headaches
* Smaller codebase size means reduced maintenance and communication costs
* Simplicity of the design means cost of change is kept low
* The explicit programming model makes codebase easier to follow and debug
* All-in-one packaging reduces cost of deployment and configuration

### Final Thoughts and Takeaways

That being said, the solution is obviously not very "enterprisy". One legitimate question is to ask is whether there is anything in the frameworks that we would be missing. For example, we could question how the solution might scale up. One the other hand, if you rely on a large framework and at some point late in development you discover that the performance -- memory consumption, reliability or anything else -- is not as good as expected, you're ... screwed. There's not much you can do except trying to rewrite parts of your application without the framework. On the contrary, if you develop your application incrementally with no framework and you keep it simple, then if at some point it's not fast enough you are in a good position for applying whatever optimization is needed.

For some teams, using large frameworks is a belief that they'd rather use a proven solution than develop their own. If chosen well, they have a tested and stable component that works. They might wonder if they have the skills and knowledge to develop their own solution. Will it work as expected? How do we know for sure? How easy can we make it evolve as our needs change? How reliable will our solution be? Do we have the skills to come up with a good API design? etc.

Another question that arises is that you probably want to avoid solving problems already taken care of. If you can find a well-tested, well-crafted library that is easy to use and addresses the problem you're trying to solve and just that problem, and if that library has minimal to no dependency, you might as well use it. Where to draw the line between doing it yourself simply and using an existing solution is a judgment call. 

For the rewrite, I ended up using only few off-the-shelf libraries -- mainly one for the web server, one for rendering HTML templates and one for parsing command line parameters. I chose libraries that I liked and that solved a single problem in a simple and elegant way -- no less, no more. When I could not find a suitable library, I followed the Do It Yourself Simply principle and coded my solution. Some problems are not hugely difficult and you can solve them easily with few lines of code. As an example, the code that replaces SiteMesh in the new version of the application is no more than 100 lines long. 

To avoid reinventing the wheel you can scan through the conventional framework code looking for special case handling that you wouldn't have thought of. Also look at how the framework solves the problems you're already aware of, to draw on as many good ideas as possible. The point is to steal the good ideas, and rewrite just the code you need in a simple way. With mastery of test-driven-development, you will be able to incrementally add as you need more. Of course, be careful to avoid turning your own solutions into large frameworks. 

One last consideration is that frameworks allow you to do so many things quickly and that's a very good point. One situation where it might be desirable to use a framework is when you don't care so much about the code, for instance to do a proof of concept for a product idea.

---

I believe there is often an illusion that frameworks will help much more than what they do in reality. You tend to want to use more of what the framework has to offer to keep your project in the same ecosystem. You often have to twist your code until for it to fit the framework. You end up paying a hefty price for the added complexity, difficulties in test automation, learning curve, keeping up to date and upgrading. All of that adds to the cost of ownership of your codebase. 

Staying away from large frameworks as much as possible will reduce your risk and increase your options. You can still add a framework later if you find you really need to. 

Simplicity is the best way to keep your agility: speed of delivery and ability to change.

> Simplicity is the ultimate sophistication. -- Leonardo da Vinci