# Notes--Versioning-Event-Sourced-System
My quick notes about the "Versioning in an Event Sourced System" book by Greg Young (from his free, online version of 
2017-06-13 at *https://leanpub.com/esversioning/read*).


This text is not to discuss Event Sourcing but to discuss how to version Event Sourced systems.

## Why Version?

Over the years, I have met many developers who run into issues dealing with versioning, particularly in Event Sourced systems. This seems odd to me. As we will discuss, Event Sourced systems are in fact easier to version than structural data in most instances, as long as you know the patterns for how to version, where they apply, and the trade-offs between the options.

## Why can’t I update an event?

It continues to amaze me that, with every architectural or data style, there always comes a central question that defines it. When looking at document databases, the question that defines them is, “How do I write these two documents in a transaction?”

This question is actually quite reasonable from the perspective of someone whose career has been spent working with SQL databases, but, at the same time, it is completely unreasonable from the perspective of document databases. If you are trying to update multiple documents in a transaction, it most likely means that your model is wrong. Once you have a sharded system, attempting to update two documents in a transaction has many trade-offs in terms of transaction coordination. It affects everything.

Similarly, there is also such a question for those coming into Event Sourcing and dealing with an Event Store: “Why can’t I update an event?”

Reasons to don't update:
* Immutability
* Consumers
* Audit log: 
  Immutability is immutable. The moment you allow a single edit, everything becomes suspect.
* WORM drives (security)
* Crime

Given all the reasons why we may not want to be able to edit an event, the question before us is: how do we handle an Event Sourced system, given such constraints? How can we run on a WORM drive while keeping a proper audit log and retain the ability to handle changes over time? How can we avoid “editing an event”?

## Basic Type Based Versioning


