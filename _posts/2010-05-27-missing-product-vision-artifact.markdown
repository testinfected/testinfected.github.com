---
layout: post
title: Product vision and Strategic Design
categories: [Scrum, DDD]
---

When I coach Scrum teams, I usually spend a significant amount of time working with the Product Owner to teach her how to create an effective **product vision**, one that will act as a guiding light for the Scrum team and stakeholders. 

I don't want to delve into the details of product envisioning activities. I usually use a number of workshops and techniques to produce some (or all) of the following artifacts:

- Elevator statement
- Product box
- Business goals 
- User goals
- Value drivers
- Key attributes and capabilities
- Risk assessments
- etc.

What I'm interested in talking about is the activity of defining the product quality attributes and how strategic design (in the Domain Driven Design sense) fits in. Thinking about the quality attributes is key to defining "done". Product quality attributes give meaning to the product being _potentially shippable_.

Among the quality attributes, some will pertain to the external quality of the product, like usability, securability or availability for instance. Others will focus on the internal quality. This is the case of maintainability, modularity, evolvability, etc. 

Yesterday, I was out for beers with my colleague Ernst Perpignand and [Greg Young](http://codebetter.com/blogs/gregyoung/). Greg was in Montreal to give his Domain Driven course and we (at [Pyxis](http://www.pyxis-tech.com)) sponsored the event by hosting the course in our Laval office. Of course during the evening we talked about CQRS, Event Sourcing, write and read models and all the stuff Greg likes to talk about. Needless to say I had a great time.

Those discussions made me realize how important it is to communicate that context maps are strategic outputs of the product vision activities. There are good reasons why Eric Evans talks about **strategic design** when he discusses bounded contexts and distillation. He refers to the conceptual core of the system as the "vision" of the system. It is indeed the guiding vision for internal quality attributes of the product. Determining our own bounded context and its relationships to others is both a political and business decision. We need to understand where the potential for ROI lies to design and architecture in consequence.
