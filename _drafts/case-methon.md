# CASE Method: Function and Process Modelling

A critical success factor in developing computer-based systems is the effective modelling of business functions and processes.

## Introduction

Function models are used to represent what a business does or needs to do, to meet it's objectives and respond to events.

- A business function is something that the business does or needs to do.
- An objective is something quantifiable (you can measure it) and is something that the business has set out to achieve. The importance of business functions is made more visible when they are held to specific objectives.
- Functions are arranged in a hierarchy with increasing detail of the lower levels; this arrangement is for convinience and can help to check for completeness (what's need to be done to achieve it's parent function).
- We describe functions in a way that is independent of any possible implementation mechanism; that is, we record **what** the function is, not **how** it is to be carried out.
- Events are happenings, often external to the business, which trigger business functions.

Function models are constructed in a manner that is independent of any implementation technique. Function models are really useful ways of clearly describing what a business does for that users, analysts and system engineers can agree on the requirement. They are also independent of mechanisms or organization, and thus can be used for many years, even if the organization changes or new mechanisms are found.

> Analyst must model **what** is done, not **how** it is done.

The aim is to produce a robust and rigorous model, free from details of the mechanisms for performing the functions. If these were included in the model they could prejudice the eventual choice of mechanisms, and would reduce the longevity of the model as it could be overtaken rapidly by technical change, even though the actual business content might still be valid.

## A simple example

There are a lot of ways of gaining information about the organization under analysis and a number of sources from which functions can be found. But **interviewing is recommended** as the primary source, with cross-checks from existing documentation, abservation, practical experience and generic hierarchies.

> A Word of Warning.
> 
> In some organizations internal politics are rife, so you need to be wary of the skilled manager (or perhaps we should use the term manipulator) deliberately feeding invalid data, which you might inadvertently use furthering the political intent of this person. Avoid getting involved in this type of activity, carefully cross-check key assertions and continually work on questioning techniques and other approaches that cut through the politics to find out what the business actually does or needs to do, as opposite to how the politics work.

At this point in an analysis you may have hundreds of pages of notes, highlighted documents, mental images, and so on. You will already have done some consolidation to arrange the information into categories such as events, functions and entities. You will also have rewritten what you have been told (or understood) into unambiguous functions, which are independent of the organization structures, roles or mechanisms. Now you must turn these into a function model.

## Identifying Business Functions

So What Have We Found?

Function models are used to represent what a business does, or needs to do, to meet its objectives and respond to events:
- A business function is something that the business does, or needs to do.
- An objective is something quantifiable (you can measure it) and is something that the business has set out to achieve. The importance of business functions is made more visible when they are tied to specific objectives.
- Functions are arranged in a hierarchy, with increasing detail at the lower levels; this is for convenience and as a navigation facility, giving a context to a function when it is being read (i.e. it needs to be done to achieve its parent function). This arrangement also helps us to check for completeness.
- We describe functions in a way that is independent of any possible implementation mechanisms; that is, we record **what** the function is, not **how** it is to be carried out.
- Events are happenings, often external to business, which trigger business functions.

Function models are constructed in a manner that is independent of any implementation technique and organizational implications.

*So function models are really useful ways of clearly describing what a business does so that users, analysts and system engineers can agree on the requirement. They are also independent of mechanisms or organization, and thus can be used for many years, even if the organization changes or new mechanisms are found.*

## The Function Hierarchy

To construct a hierarchy from scratch, we start with initial interview notes and known business objectives and build a framework showing the essence of the business top down. The hierarchy is cross-checked and completed bottom up by taking detailed functions and fitting them into an appropriate place in the hierarchy, which will grow, evolve and change shape as the exercise proceeds.

Of the six interrogative words, who, what, when, where, how and why, only the what and why should appear in a function model; the **what** is an operation the business does, or needs to do, and the **why** is the objective it is trying to achieve by doing it. Of the others: **who** is extracted with the removal of jobs and roles; **when** does not appear because functions have no implicit sequence or timing in the hierarchy; **where** is removed when references to parts of the organization are taken out; and **how** is separated out with the removal of the mechanisms.

## Introducing Function Dependency

The concept of a function dependency recognizes that a function may have to wait on the completion of one or more other functions before it can be carried out itself. This dependency information can help us make decisions on such things as:
- the scope of a system,
- the order in which systems can be developed,
- the order in which functions within a system can be represented to the users of that system.

## Function Dependency Diagrams

Businesses everywhere spend much of their time reacting to external stimuli. Function dependency diagrams are an easy-to-understand way of modelling what happens on receipt of an event, what functions are triggered and what key results are eventually achieved. The diagrams can be used for rigorously defining sequence, interdependency and functions that can be carried out in parallel at a business, system or program/procedure level. Diagrams at the high level are very good for agreeing the scope and sequencing of new systems with management. At the detailed level they must be used on leaf (lowest level) functions to maintain accuracy and are a useful modelling technique for functions that are oriented towards key results following external stimulation.

Function dependency diagramming is another excellent technique to learn to help model the business requirement, communicate with users and build appropriate systems.

## Related Concepts

Common functions - when the same function appears in more than one place in a function hierarchy. They can be removed by escalating to a common parent, or recorded and cross-referenced by choosing one as a master. The frequency and urgency of a function can be recorded against each of organization, role, georgraphy and time period to guide a system designer towards sensible options for supporting the business in question.

The interaction between business functions and the information model is important, we have to pay particular notice to whether a function creates, retrieves, updates or deletes information. A CASE tool is becoming more and more essential to handle the vast scale of important cross-related information.

## Detailed Function Definition

An elementary function is a function that 'publishes' the results of its actions in one go, or leaves the state of the information known to the business as it was before it started. Function logic is a way of defining precisely the information to be collected from the information model (by navigating around it), the processing to be carried out on that information, and the scope of any modifications to be made. A number of languages, with varying levels of rigour, are available for defining function logic.

If the systems department is to be an integral part of your business and thus contribute towards its profitability, the exercise of defining functions and building systems **must** be done in a pragmatic and cost-effective way. Too often we have seen **purists** take elapsed years to define the detail requirement unambiguously prior to starting system design, frequently without user involvement. Meanwhile the business requirement may itself have changed!

> Don't be like the person who buys an old house and wants it completely restored and perfect before inviting friends to dinner. By the time perfection is achieved your friends may have given up on you, moved house themselves, or even left the country.

In general it is a far better to get the balance right and define requirements in sufficient detail to enable the designers and implementors to make a good job of it. Mistakes **will** occur later, but prototyping and iteration based on a good starting poing are cost-effective and timely ways of achieving the desired result.

Logical System Design
Dataflow Diagrams
Realtime Modelling
Physical System Design
When to Use What