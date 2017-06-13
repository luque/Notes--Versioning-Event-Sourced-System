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

A new version of an event must be convertible from the old version of the event. If not, it is not a new version of the event but rather a new event.

What happens when we have a Blue-Green or similar deployment where there are multiple concurrent versions of the software running side by side? If we upgrade node A to be version 2 here and node B is still on version 1, can node B read from the event stream that node A has written a version 2 event? Unfortunately, using this style of serialization requires every node to actually have the type for the event, as the type describes the schema of the event. If a consumer does not have the type, it will not be able to even deserialize that event.

There is a common way in which people try to avoid the issues that arise from having old subscribers that do not understand the new version of the event. The idea is to have the new version of the producer write both the _v1 and the _v2 versions of the event when it writes. This is also known as Double Publish

A rule must be followed in order to make this work. At any given point in time, you must only handle the version of the event you understand and ignore all others.

Over time, the old version of the event will be deprecated. This gives subscribers time to catch up.

You should generally avoid versioning your system via types in this way. Using types to specify schema leads to the inevitable conclusion that all consumers must be updated to understand the schema before a producer is. While this may seem reasonable when you have three consumers, this completely falls apart when you have 300.

Accepting the constraint that our serializer has absolutely no versioning support seems an odd place to start when almost all serializers have some level of versioning support (json, xml, protobufs, etc.). Accordingly, let’s try removing this constraint.

## Weak Schema

The serialization formats discussed up until now operate based on what is known as strong schema. In the type example for the .NET and binary serializer, the schema is stored in the type. This is also known as out-of-band schema, meaning that the schema is not included with the message but is held outside it.

The problem with strong schema, especially when using out-of-band schema such as types, is that without the schema you will not be able to deserialize a given message. This leads to the previously described problem of needing to update a consumer before updating a producer, which is unacceptable in many situations, as the consumer will not be able to deserialize the message otherwise.

Most systems today do not use this method of serialization for exactly these reasons. Instead, they will use something like json or xml, combined with what is known as weak-schema or hybrid-schema, to serialize their messages. While this entails more rules that must be followed, it also offers more flexibility, providing the rules are followed.

What if instead of deserializing to a type it was instead mapped to? The rules for mapping are simple. When mapping, you look at the json and at the instance.

*    Exists on json and instance -> value from json
*    Exists on json but not on instance -> NOP
*    Exists on instance but not in json -> default value

When using mapping, there is no longer an addition of a new version of the event. Instead, you just edit the event already in place.

The mapping handles the rest. If you have an InventoryItemDeactivated in the first version and you map it to one expecting the second version, it will still work, but Reason will be set to a default value.

There are, however, two factors that must be remembered here.

The first is that you are no longer allowed to rename something. 

The second is that there will often be programmatic checks to ensure what you expect to be in the message after the mapping is in fact present

Another option is to use hybrid-schema, where some things are required and some things are not, the latter being treated as being mapped. Protobufs is an example of a format that supports this. The general rule of thumb when working with hybrid-schema is to make the things without which the event would make absolutely no sense a requirement while leaving everything else as optional. 



