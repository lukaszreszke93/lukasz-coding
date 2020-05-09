"4+1" Architectural View Model
===============================

Trying to capture the software system's architecture into one diagram can be hard. It might be even impossible for a complex system if there would be a readability requirement.
One reason for that is the software systems have multiple layers that should be represented on those diagrams. So instead of trying to create one big diagram, it should be easier to split it into multiple system diagrams.

"4+1" Architectural View Model created by Phillippe Kruchten, splits the system diagrams into 5 *architectural views*.

Those views are represented by different stakeholders.
Some of them are going to be interested only in the business logic.

Others will be interested in how the code should be written.

The hosting of the application will be important for other people.

Security and other non-functional requirements?
And much more details, important to different departments.

## Architectural Model
According to this model, there are 5 views working with each other:
- logical view,
- process view,
- development view,
- physical view,
- scenarios

The "4+1" View Model diagram

![4+1 architectural view model diagram](https://lukaszcoding.com/wp-content/uploads/2020/05/41-architectural-view-model-diagram.png)

Each of these views has a different goal and will be addressed when talking to different stakeholders. The goal is to be able to distinguish which view should be used in a given discussion or meeting. 

I am not sure yet what person in the organization should be 'the glue' of those views. I haven't used that yet. It's a new concept for me which I am trying to understand and start *implementing*. There's a project on a horizon where there will be a chance to gather requirements and design a system from scratch. As I see it, it will be a good time for discussions, that are more aware and focused on different views. 

Different architectural models will also differ with the notation that is used to represent them.

### The Logical View
When thinking about the logical view, it should be focused on the business value we're trying to provide. This is also described as end-user functionality.

This architectural view can be very useful when talking about and looking for service boundaries. The focus should go on describing the business problem that we want to solve. The problem domain if you will. The logical view shows system as decomposition of specific functionality areas.

### The Process View
Process view is focused on communication between services and processes. It also considers non-functional requirements. Those are:
 - performance,
 - availability,
 - issues of concurrency and distribution,
 - fault-tolerance,
 - scalability
and probably a few more.

This view also should try to describe how the logical view will fit into all listed considerations.

The main purpose of the process view (and its diagrams) is to picture how the pieces of information will be exchanged between business entities.


### The Development View
This view presents the system from the developer's perspective. One of the focuses is the organization of software modules in the development environment. This organization includes file and folder structure of the codebase, for example. The development view is also the place where the team should discuss it's organization, coding standards, testing strategy, and tools. All the things we love the most in writing code 🤗

The development view should also be considered when thinking about a specific domain's architectural style.
Should it be onion? Hexagonal? Or perhaps something simpler?

### The Physical View
*Mapping the software to the hardware*. This view is taking into consideration how the services should be deployed, how the performance can be increased by scaling the application.

Reliability, availability, and other non-functional requirements are also taken into consideration here.

You probably noticed that some of the mentioned non-functional requirements are duplicated in the process view... but that's exactly the point.

The process view is focusing on the software side of handling those problems, whereas the physical view is looking from a hardware perspective.

Stakeholders describing the business problems will be probably least interested in this view.

### The "+1" View
It's been well described by the author by *"Putting it all together"*.

This view is called scenarios or use cases.

It's overlapping with other views, but it's all that the user cares about as they describe sequences of interactions between system entities to solve business problem that user wants to solve.

It's used to illustrate and validate the architecture design. It helps to 'tie' the architecture together to solve the business problem.

## Summary
Those views are supposed to help us build greater software. Use them wherever you find it useful. It might click instantly in some teams to adopt the "4+1" View Model 1:1. Others might just want to take a part of it.

I think it is worth being aware of that and use it when appropriate.

Example use-case situation could be when you have a feeling that discussion with business stakeholders about the problem domain is going into code workarounds or design decisions. Whereas it should go into the business process description that will solve that problem.

I am willing to try this approach, perhaps modernize it a bit and see how it works for me.