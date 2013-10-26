---
layout: blog-post
shortenedlink: Towards universal event analytics - building an event grammar
title: Towards universal event analytics - building an event grammar
tags: event analytics grammar model
author: Alex
category: Inside the Plow
---

As we outgrow our "fat table" structure for Snowplow events in Redshift, we have been spending more time thinking about how we can model digital events in Snowplow in the most universal, flexible and future-proof way possible.

When we blogged about [building out the Snowplow event model] [event-model-post] earlier this year, a comment left on that post by [Loïc Dias Da Silva] [loic] made us realize that we were missing an even more fundamental point: defining a Snowplow event **grammar** to underpin our Snowplow event dictionary. Here is part of Loïc's excellent comment - although I would encourage you to read it in full [on the blog post] [event-model-post]:

_Hi, we're also working on an event model for our global eventing platform but our events currently are more macro, inspired by RDF in a sense:_

_An Actor(id/type) made and Action(verb, context) on another Object(id/type)._

_Each Actor, Action and Object can hold k/v properties._

_The context itself, owned by the action, is a k/v dictionary._

So in designing his event grammar, Loïc was influenced by the [Resource Description Framework] [rdf], the W3C specifications for modelling relationships to web resources.

An event grammar inspired by RDF is certainly interesting, but I am using a much older, more sophisticated and more tested "event grammar" to write this sentence: the **grammar of human language**. Why not start, then, from the core grammar underpinning English, Latin, Greek, German and other languages to see just how far this approach can take us in modelling events in the digital world?

So, in the rest of this post we will:

1. [Introduce the components of our grammar](/blog/2013/08/12/towards-universal-event-analytics-building-an-event-grammar#grammar)
2. [Model some ecommerce events](/blog/2013/08/12/towards-universal-event-analytics-building-an-event-grammar#ecommerce)
3. [Model some videogame events](/blog/2013/08/12/towards-universal-event-analytics-building-an-event-grammar#videogame)
4. [Model some digital media events](/blog/2013/08/12/towards-universal-event-analytics-building-an-event-grammar#media)
5. [Discuss what we have learnt](/blog/2013/08/12/towards-universal-event-analytics-building-an-event-grammar#learnings)
6. [Draw some conclusions](/blog/2013/08/12/towards-universal-event-analytics-building-an-event-grammar#conc)

<!--more-->

<a name="grammar"> </a>
<h2>1. The components of our grammar</h2>

All of the human languages mentioned above (and many, many others) share the same fundamental building blocks in their grammars for describing an event with a verb in the _active voice_:

![grammar] [grammar]

To go through these in turn:

* **Subject**, or noun in the _nominative_ case. This is the entity which is carrying out the action: "**I** wrote a letter"
* **Verb**, this describes the action being done by the Subject: "I **wrote** a letter"
* **Direct Object**, or simply _Object_ or noun in the _accusative_ case. This is the entity to which the action is being done: "I wrote **a letter**"
* **Indirect Object**, or noun in the _dative_ case. A slightly more tricky concept: this is the entity indirectly affected by the action: "I sent the letter _to_ **Tom**"
* **Prepositional Object**. An object introduced by a preposition (in, for, of etc), but not the direct or indirect object: "I put the letter _in_ **an envelope**". In a language such as German, prepositional objects will be found in the _accusative_, _dative_ or _genitive_ case depending on the preposition used
* **Context**. Not a grammatical term, but we will use context to describe the phrases of time, manner, place and so on which provide additional information about the action being performed: "I posted the letter **on Tuesday from Boston**"

With these grammatical building blocks defined, let's now put them through their paces modelling some digital events - starting with some online retail events:

<a name="ecommerce"> </a>
<h2>2. Modelling some ecommerce events</h2>

Here are some ecommerce events mapped to our grammatical model:

![ecomm1] [ecomm1]

In this event, a shopper (Subject) views (Verb) a t-shirt (Direct Object) while browsing an online store (Context).

![ecomm2] [ecomm2]

Here we introduce an Indirect Object which has been affected by the event: the shopper (Subject) adds (Verb) a t-shirt (Direct Object) to her shopping basket (Indirect Object). Again, this is while browsing the online store (Context).

![ecomm3] [ecomm3]

Here we have an Object introduced by preposition: the shopper (Subject) pays (Verb) for his order (Prepositional Object). This is all within the checkout flow (Context).

<a name="videogame"> </a>
<h2>3. Modelling some videogame events</h2>

So far so good, but how well does this model work with events generated by a gaming session?

![videogame1] [videogame1]

In a gifting screen within the game (Context), the player (Subject) gifts (Verb) some gold (Direct Object) to another player (Indirect Object).

![videogame2] [videogame2]

During a two-player skirmish (Context), the first player (Subject) kills (Verb) the second player (Direct Object) using a nailgun (Prepositional Object). This illustrates how your end-users can be the Object of events, not just their Subjects.

![videogame3] [videogame3]

Here we illustrate a reflexive verb: through grinding (Context), the player (Subject) levels herself up (Verb, reflexive). A reflexive Verb is one where the Subject and the Object are the same.

<a name="media"> </a>
<h2>4. Modelling some digital media events</h2>

This seems to be working well! Finally, let's map our new event grammar onto the world of digital media and publishing:

![media1] [media1]

While consuming media on your site (Context), a user (Subject) reads (Verb) an article (Direct Object).

![media2] [media2]

Wanting to share content socially (Context), a user (Subject) shares (Verb) a video (Direct Object) on Twitter (Prepositional Object). Also note that Twitter here is a proper noun (not a common noun).

![media3] [media3]

Working from the moderation UI (Context), an administrator (Subject) bans (Verb) user #23 (Direct Object). This illustrates how an end-user can be the Object of an event, and how someone other than an end-user can be the Subject of the event.

<a name="learnings"> </a>
<h2>5. What have we learnt</h2>

As you can see, it is relatively straightforward to map any of the digital events above into these six "slots" of: Subject, Verb, Object, Indirect Object, Prepositional Object and Context. This is unsurprising: our core grammar has been unambiguously describing events in many different human languages across thousands of years.

Going through the above exercise, several further things have become clear to us that we will want to factor into the Snowplow event grammar going forwards:

### Implicit Subjects are a mistake

Most web and event analytics systems make the mistake of making the Subject of the event implicit:

    (End user) adds product to basket
    (Admin) bans user #23

This is a mistake, because as we have seen above, expressing the Subject is a key component of our event grammar.

Going further, it is particularly dangerous to assume that the Subject of every event is your end-user or customer, because we have seen cases where this is not the case.

### An entity can be Subject or Object or both across multiple events

As per these gaming examples:

    User #1 gifts gold to user #2
    User #2 kills user #3
    User #2 levels up
    Admin bans user #1

As we can see from this, the same entities will be found as Subject, Direct Object, Indirect Object or Prepositional Object depending on the event.

Most analytics systems miss the fact that an end-user (for example) is not merely the implicit Subject of multiple events, but is in fact an entity which is the Subject and the Object of different events.

### We can keep our Verbs really simple

All of the events above were modelled simply using verbs in the _active voice_, not the _passive voice_:

* Active voice: "I watch a video"
* Passive voice: "the video was watched by Alex"

We don't need to use passive voice for our event model, because we can always derive (if needed) a passive voice event from our active voice event.

Going further, Verbs conjugate in lots of other ways (tense, person, mood etc) - but again we don't need to include any of this into our event model: all of this can be derived (if needed) from our event's Context.

### Context is king

Our idea of Context does not map cleanly onto a singular grammatical component, but it is just too useful to exclude. In fact, de facto we already have a rich web context for Snowplow events in our [Canonical event model] [canonical-event-model], including:

* When the event occurred
* Where (geographically) the event occurred
* Properties of the device on which the event occurred

<a name="learnings"> </a>
<h2>6. Conclusions</h2>

We hope this has been an interesting exploration of how we can potentially adapt and simplify the grammar of human languages to express a new grammar for digital events. We are really excited about the possibilities this opens up - initially around expressing such a grammar in our new [Avro] [avro] event model, and later hopefully in graph databases such as [Neo4J] [neo4j].

Of course, we have only just started to sketch out this new event model, and we hope that it will prompt a wider debate with the Snowplow and analytics communities. We are excited to evolve these ideas and build a model for universal event analytics with you, together - and we look forward to continuing the conversation on our [snowplow-user mailing list] [snowplow-user].

And finally, many thanks again to [Loïc Dias Da Silva] [loic] for sharing his original Actor-Action-Object idea on our blog!

[event-model-post]: http://snowplowanalytics.com/blog/2013/02/04/help-us-build-out-the-snowplow-event-model/
[loic]: https://twitter.com/mglcel
[rdf]: http://en.wikipedia.org/wiki/Resource_Description_Framework

[grammar]: /static/img/blog/2013/08/event-grammar.png

[ecomm1]: /static/img/blog/2013/08/grammar-ecomm1.png
[ecomm2]: /static/img/blog/2013/08/grammar-ecomm2.png
[ecomm3]: /static/img/blog/2013/08/grammar-ecomm3.png

[videogame1]: /static/img/blog/2013/08/grammar-videogame1.png
[videogame2]: /static/img/blog/2013/08/grammar-videogame2.png
[videogame3]: /static/img/blog/2013/08/grammar-videogame3.png

[media1]: /static/img/blog/2013/08/grammar-media1.png
[media2]: /static/img/blog/2013/08/grammar-media2.png
[media3]: /static/img/blog/2013/08/grammar-media3.png

[canonical-event-model]: https://github.com/snowplow/snowplow/wiki/canonical-event-model

[snowplow-user]: https://groups.google.com/d/forum/snowplow-user
[avro]: http://avro.apache.org/
[neo4j]: http://www.neo4j.org/