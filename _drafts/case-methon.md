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

A simple example
Basic Conventions and Definitions
Identifying Business Functions
The Function Hierarchy
Introducing Function Dependency
Function Dependency Diagrams
Related Concepts
Detailed Function Definition
Logical System Design
Dataflow Diagrams
Realtime Modelling
Physical System Design
When to Use What