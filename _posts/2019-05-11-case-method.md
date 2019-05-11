---
layout: post
title:  "CASE Method: Function and Process Modelling"
date:   2019-05-11 12:26:00 +0300
categories: software architecture
---
# CASE Method: Function and Process Modelling

![](/assets/img/case-method/tree.jpg)

A critical success factor in developing computer-based systems is the effective modelling of business functions and processes.

## Introduction

Function models are used to represent what a business does or needs to do, to meet it's objectives and respond to events.

- **A business function** is something that the business does or needs to do.
- **An objective** is something quantifiable (you can measure it) and is something that the business has set out to achieve. The importance of business functions is made more visible when they are held to specific objectives.
- Functions are arranged in **a hierarchy**, with increasing detail of the lower levels; this is for convenience and as a navigation facility, giving a context to a function when it is being read (i.e. it needs to be done to achieve its parent function). This arrangement also helps us to check for completeness.
- **Events** are happenings, often external to the business, which trigger business functions.

Function models are constructed in a manner that is independent of any possible implementation mechanism or organization, and thus can be used for many years, even if the organization changes or new mechanisms are found. That is, we record **what** the function is, not **how** it is to be carried out.

> Analyst must model what is done, not how it is done.

The aim is to produce a robust and rigorous model, free from details of the mechanisms for performing the functions. If these were included in the model they could prejudice the eventual choice of mechanisms, and would reduce the longevity of the model as it could be overtaken rapidly by technical change, even though the actual business content might still be valid.

The interaction between business functions and the information model is important, we have to pay particular notice to whether a function creates, retrieves, updates or deletes information. A CASE tool is becoming more and more essential to handle the vast scale of important cross-related information.

So function models are really useful ways of clearly describing what a business does so that users, analysts and system engineers can agree on the requirement. They are also independent of mechanisms or organization, and thus can be used for many years, even if the organization changes or new mechanisms are found.

## Information Gaining

There are a lot of ways of gaining information about the organization under analysis and a number of sources from which functions can be found. But **interviewing is recommended** as the primary source, with cross-checks from existing documentation, abservation, practical experience and generic hierarchies.

> A Word of Warning.
>
> In some organizations internal politics are rife, so you need to be wary of the skilled manager (or perhaps we should use the term manipulator) deliberately feeding invalid data, which you might inadvertently use furthering the political intent of this person. Avoid getting involved in this type of activity, carefully cross-check key assertions and continually work on questioning techniques and other approaches that cut through the politics to find out what the business actually does or needs to do, as opposite to how the politics work.

At this point in an analysis you may have hundreds of pages of notes, highlighted documents, mental images, and so on. You will already have done some consolidation to arrange the information into categories such as events, functions and entities. You will also have rewritten what you have been told (or understood) into unambiguous functions, which are independent of the organization structures, roles or mechanisms. Now you must turn these into a function model.

## The Function Hierarchy

To construct a hierarchy from scratch, we start with initial interview notes and known business objectives and build a framework showing the essence of the business top down. The hierarchy is cross-checked and completed bottom up by taking detailed functions and fitting them into an appropriate place in the hierarchy, which will grow, evolve and change shape as the exercise proceeds.

**Common functions** - when the same function appears in more than one place in a function hierarchy. They can be removed by escalating to a common parent, or recorded and cross-referenced by choosing one as a master. The frequency and urgency of a function can be recorded against each of organization, role, georgraphy and time period to guide a system designer towards sensible options for supporting the business in question.

Of the six interrogative words, who, what, when, where, how and why, only the what and why should appear in a function model; the **what** is an operation the business does, or needs to do, and the **why** is the objective it is trying to achieve by doing it. Of the others:
- **who** is extracted with the removal of jobs and roles;
- **when** does not appear because functions have no implicit sequence or timing in the hierarchy;
- **where** is removed when references to parts of the organization are taken out; and
- **how** is separated out with the removal of the mechanisms.

## The Function Dependency

The concept of **a function dependency** recognizes that a function may have to wait on the completion of one or more other functions before it can be carried out itself. This dependency information can help us make decisions on such things as:
- the scope of a system,
- the order in which systems can be developed,
- the order in which functions within a system can be represented to the users of that system.

### Function Dependency Diagrams

Businesses everywhere spend much of their time reacting to external stimuli. **Function dependency diagrams** are an easy-to-understand way of modelling what happens on receipt of an event, what functions are triggered and what key results are eventually achieved. The diagrams can be used for rigorously defining sequence, interdependency and functions that can be carried out in parallel at a business, system or program/procedure level. Diagrams at the high level are very good for agreeing the scope and sequencing of new systems with management. At the detailed level they must be used on leaf (lowest level) functions to maintain accuracy and are a useful modelling technique for functions that are oriented towards key results following external stimulation.

Function dependency diagramming is another excellent technique to learn to help model the business requirement, communicate with users and build appropriate systems.

## Detailed Function Definition

**An elementary function** is a function that *publishes* the results of its actions in one go, or leaves the state of the information known to the business as it was before it started.

**Function logic** is a way of defining precisely the information to be collected from the information model (by navigating around it), the processing to be carried out on that information, and the scope of any modifications to be made. A number of languages, with varying levels of rigour, are available for defining function logic.

If the systems department is to be an integral part of your business and thus contribute towards its profitability, the exercise of defining functions and building systems **must** be done in a pragmatic and cost-effective way. Too often we have seen **purists** take elapsed years to define the detail requirement unambiguously prior to starting system design, frequently without user involvement. Meanwhile the business requirement may itself have changed!

> Don't be like the person who buys an old house and wants it completely restored and perfect before inviting friends to dinner. By the time perfection is achieved your friends may have given up on you, moved house themselves, or even left the country.

In general it is a far better to get the balance right and define requirements in sufficient detail to enable the designers and implementors to make a good job of it. Mistakes **will** occur later, but prototyping and iteration based on a good starting point are cost-effective and timely ways of achieving the desired result.

## Logical System Design

## Dataflow Diagrams

**The datastore** represents a persistent data storage capability and is expressed in terms of entities, attributes and relationships from the data model of the system.

**The dataflow** shows a movement of data (again in terms of entities, attributes and relationships) between two processes, between a process and a datastore, or between a process and an external entity.

Dataflows are called **balanced** if they are into and out of processes that are not further decomposed. The task of decomposing a process and balancing its dataflows, by assigning them to its sub-processes, is a top-down design activity which, if taken to a low enough level of detail, can often form an adequate specification of a process from which final construction can take place (using a computer language, manual procedure, electronic circuit, etc.).

**The dataflow diagramming** technique is, like many others, an iterative, evolutionary approach to arriving at the final result. Many mistakes will be made and repaired along the way to a completed system definition.

## Realtime Modelling

Realtime modelling can be done using **state transition diagramming**, which is another useful technique for modelling systems where it is important to know what happens when different events occur, dependent on the state of the system. These tend to be critical control systems, so it is important to be able to check them out with the user and using various quality checks. Realtime sub-systems are an essential component of many computer systems today, and become more and more important whenever process control and interactive graphics user interfaces are considered.

State transition diagrams give us the ability to model a system from the viewpoint of the system being in a given state, going through a transition to another state following an event, and possibly carrying out some consequential process. They also allow us to model interrelated realtime sub-systems and handle queues and concurrent processes, and are particularly appropriate for process control systems and interactive graphical user interfaces.

## Physical System Design

Designing a system to a level of detail necessary for it to be built successfully demands design work to be done with knowledge of the physical implementation language, machinery or other implementation vehicle. The goal of the functional design activity is to design computerized systems that are:
- relevant,
- usable, and
- flexible.

Techniques such as decomposition, dependency and dataflow diagrams provide a good starting point for a physical design, and decomposition, in the form of a structure chart, can be used to record the results of a physical design for a procedural computer programming language.

## A Final Analogy

As a final reminder about the importance of proper analysis and preparation throughout all the stages of creating information systems, let us consider the analogy of decorating a room in your home.

It is spring and you get the urge to redecorate, so you rush out to the hardware shop and buy lots of lovely paint, wallpaper, new paint brushes, and so on. Then you come home and at least you have the sense to move the furniture out of the room or at least cover it up while you are working. A quick cup of tea (after all that rushing about) and you set to with the paint brush (it really is such fun painting you can't wait to get to work). Result: you run out of paint, or you have loads left over, it is badly done, the cracks and the holes and the old flaking paint still show through, your children hate the colour!

The *fun* bit at the end will only be successful if you get the requirements and the design right - what colour, what type of paint, how much, will you still like the pattern in three years' time? Then you must construct the system properly. Decide on the order of the preparation work: sanding the paintwork, preparing the walls, washing them, filling the cracks, and so on. Work out the order for the actual physical design: which bit will be painted first, how many coats of paint, how long between coats, and so on.

Now you can actually start coding (painting).

# Sources

- **Richard Barker, Cliff Longman.** Case*Method: Function and Process Modelling.
