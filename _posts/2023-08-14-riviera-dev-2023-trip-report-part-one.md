---
layout: post
title: Riviera DEV 2023 Trip Report, Part One
date: 2023-08-14 01:59 +0200
published: true
---

Riviera DEV 2023 was my first Dev conference. It's held in the French Riviera, it has a great atmosphere, exceptional food and provides no excuses for you not to attend. For all those reasons, and secondary because I was one of the speakers and I am leaving at a few bus stops, I attended day 2 and 3 of the conference. This is my **Riviera DEV 2023 Trip Report, Part One**. There is also a [Part Two]({% post_url 2023-08-26-riviera-dev-2023-trip-report-part-two %}).

## FoundationDB: the best kept secret of new distributed architectures - Pierre Zemb

Original title: _FoundationDB : le secret le mieux gardé des nouvelles architectures distribuées !_

<i class="fas fa-circle-info fa-sm" style="color: gray;"></i> I took lots of notes during this talk, and I have more details in a [dedicated post](/posts/2023-08-27-foundationdb-the-best-kept-secret-of-new-distributed-architectures-talk-report). But here is a TLDR.

![FoundationDB](https://pierrezemb.fr/posts/notes-about-foundationdb/images/fdb-white.jpg){: .right .w-50}

FoundationDB is a distributed DB engine which lacks of many things that one may expect. Usually when you talk about an open source DB, you get a product that has everything. FoundationDB only has everything you need to start building a highly performant distributed DB. It does not impose you any Data Models, nor language. It even lacks authentication.

It's at the _foundation_  of Snowflakes and Apple iCloud, and it was Apple who bought it and made it open source. Before that FoundationDB was developed by the homonymous company, which first proceeded at creating a simulator, even before the database engine itself. The simulator is able to inject errors, like disk and network errors, and each simulation can be replayed deterministically.

Transactions are handled in a clever and practical way. Since it's distributed you can't really block data. Instead it keeps track of what changed and if necessary replays the whole transaction.

One cannot really use FoundationDB as is, and it's unlikely that a single developer could do anything with it. It's more for companies with special use-cases who want to build their own solution.

The talk was delivered [on 2023-07-11 by Pierre Zemb of Clever Cloud](https://rivieradev.fr/session/1158). It was in French.

You can read more in [<i class="fas fa-link fa-sm" style="color: gray;"></i> his post Notes about FoundationDB](https://pierrezemb.fr/posts/notes-about-foundationdb/) and in [my notes]({% post_url 2023-08-27-foundationdb-the-best-kept-secret-of-new-distributed-architectures-talk-report %}).

## A Kafka Client’s Request: There and Back Again - Danica Fine

![Kafka](https://upload.wikimedia.org/wikipedia/commons/thumb/5/53/Apache_kafka_wordtype.svg/640px-Apache_kafka_wordtype.svg.png){: .right .w-25}

This talk is a behind the scene in Kafka of what happens when you push an event. It's important to know what happens because:
- there are a lot of things happening even before the event leaves for the broker
- one must know which parameter and values to investigate when **something goes wrong** or when **performances are not good**.

<i class="fas fa-circle-info fa-sm" style="color: gray;"></i> I have written more in detail about this in a [dedicated post]({% post_url 2023-08-27-a-kafka-client-s-request-there-and-back-again %}), but if you really want to retain something, keep an eye on some parameters and values like:

- `key.serializer` and `value.serializer`, if you think there is a problem with serialization of your message and remember that kafka only thinks in _bytes_;
- `batch.size`, `linger.ms` and `buffer.memory` if you feel your messages are sent late or you are using too much bandwidth
- `max.request.size` and `request.timeout.ms` when your producer gets errors
- `request.required.acks`, `max.in.flight.requests.per.connection` and `enable.idempotence` if you have find the right balance between too much network traffic and a producer that may send many requests or duplicates
- monitor `ResponseQueueSize` and `ResponseQueueTimeMs` when you feel your cluster is not keeping up

The talk was delivered by [<i class="fa-brands fa-twitter fa-sm" style="color: gray;"/>Danica Fine](https://twitter.com/TheDanicaFine), developer advocate at Confluent. Unfortunately it was not recorded, but you can find the [same talk](https://www.youtube.com/watch?v=DkYNfb5-L9o) at other conferences. The slides are [available on Slideshare](https://www.slideshare.net/HostedbyConfluent/a-kafka-clients-request-there-and-back-again-with-danica-fine).
<!-- {% include embed/youtube.html id='DkYNfb5-L9o' %}{: .shadow .w-75} -->

## What's new in Quarkus' gRPC? - Aleš Justin

I have to say, I didn't understand everything in this talk. I have never played with Quarkus, although some of my close colleagues are using it with _mostly_ positive experience feedbacks.

To measure how much I lacked context, at a certain point I heard _Mutiny ftw_: I didn't know that Mutiny is a fairly popular library for Quarkus, and without that I didn't now why it was good at that moment of the talk.

So, more than content, what this talk gave me are some good questions about the possibilities that one has with Quarkus and gRPC. Two main points:

1. It seems there is a dedicated protocol buffer generator for Quarkus. 
2. There is a lot less code to write thanks to _injectors_, although I am not fully sure what those injectors are.

I will definitely look further into this topic if I need to get my hands dirty with Quarkus, which will probably happen in the coming months.

Thanks to Aleš Justin[^1] of RedHat for the talk. You'll find the link to the slides [here](https://rivieradev.fr/session/1106). 

## A Story about Serverless Camels Living in Containers - Kevin Dubois

Again a talk where I lacked context. On purpose. Because I like challenging myself. And because I wanted to have a look at Quarkus before the conference, but just didn't have the time. But this time more than the lack of context could the amazing other Quarkus features that were showed off during the talk.

There is this thing called Apache Camel, which does a lot of things. And there is this other thing called Apache Camel K, which for what I understood is a sub-project of Camel, and it's aimed at bringing serverless to kubernetes, hence the title "Camels Living in Containers". It's made to work with "Cloud Events" as defined in the CNCF[^2]. So far so good.

Then this happened: just by running `quarkus dev` and with the right plugins and settings, Quarkus launched by itself Postgres and Kafka. There are already plugins to bring everything together, like the `camel-quarkus-kafka`. To automatic launch kafka, you only need to list the topics you want in the application properties. I understand for many people this is something normal, but please be respectful: there are people who suffer every day because of the lack of tools for the C++ world.

More info about the talk here. Thanks to [<i class="fa-brands fa-twitter fa-sm" style="color: gray;"/>Kevin Dubois](https://twitter.com/kevindubois) of RedHat who was nice enough to explain me some of those wonders even if not related to the talk.

## Where to store your data in 2023? - Ciryl Gambis

Original Title: _Où stocker ses données en 2023 ?_

A good recap of the current database landscape, with a bit of history to explain how we got here.

The question being: what database should I choose for my data, the answer is inevitably: it depends.

<i class="fas fa-circle-info fa-sm" style="color: gray;"></i> Given my current interests, I have [more detailed notes]({% post_url 2023-08-27-where-to-store-your-data-in-2023 %}) on this talk.

The talk goes through the history of DB, from the relational introduction to the advent of NoSQL and the "Relational Strikes Back" with the NewSQL. The main points being: 

- the need for being distributed paved the road to NoSQL and new Data Models
- relational database are borrowing some tricks from NoSQL
- we have more choice than ever, so "choose wisely"

One of the talks I enjoyed the most, also because of my current role. I would personally recommend going through it to get a picture of the current DB landscape. The good being: there are many choices and it would be a pity to choose one solution without considering what's best for the data model you will work with.

More details and the slides (mostly in English) can be found [here](https://rivieradev.fr/session/1129) and [here](https://docs.google.com/presentation/d/1gIdJ18m7xE0h9XGQmZJT81mJl0jmYtia35do1NdNnSs/). The talk was delivered by Ciryl Gambis, Staff Software Engineer at Decathlon.

## The artificial intelligence explained in 25 minutes - Stéphane Philippart

Original Title: _L'intelligence artificielle expliquée en 25 minutes !_

A quick overview of what the AI is, going quickly through the categories (machine learning, narrow intelligence, general intelligence, super intelligence which I understand has been superseeded by a better concept). Stéphane Philippart proceedeed to explain the different machine learning familites: supervised, unsupervised, reinforced, with some examples in NLP, Computer Vision and Generative.

And then some examples about reusing AI models for specialized use cases, like the Object Detection Model YOLO that can be adapted to detect plastic bottles without any changes.

And then why AI is just math and stats fuelled by data that need to be clean and labeled.

And that there are many frameworks to play with, obviously in Python.

And that in 25 minutes you cannot really explain how things work, just what they are.

[Talk](https://rivieradev.fr/session/1143) by Stéphane Philippart. [Slides here](https://noti.st/philippart-s/wbASmn/lintelligence-artificielle-expliquee-en-25-minutes).

## Manage the drifts of resources Terraform with GitOps - Katia Himeur

Original title: _Gérer les drifts des ressources Terraform grâce à la méthode GitOps_

So, let's say that you describe your infrastructure using Terraform, aka "Infrastructure as Code". The you realize that there is a difference between what you described in your code and what you observed. That is a _drift_, almost an "Expectation vs Reality" meme.

All because some some human has done manual modifications on the infrastructure which were not mirrored back in the code. The solution proposed is to use [Flux](https://fluxcd.io/flux/get-started/) to _gitopsify_ (the term is mine) your terraform. Which surprised me, because I though that Terraform was already _gitopsified_.

#### Personal note
Before this talk I had a vague idea of what Terraform is, I knew well what we intend for GitOps, I had never heard of _drifts_.

After this talk, I understood I knew even less about Terraform, but I am very convinced that drifts shouldn't exist. The existence of this talk is a very bad sign and makes me very glad I'm not into devops.

This [25 minutes talk](https://rivieradev.fr/session/1156) was delivered by [<i class="fa-brands fa-twitter fa-sm" style="color: gray;"/>Katia Himeur](https://twitter.com/katia_tal), co-founder at Cockpit.io. More details and slides will be found [here](https://rivieradev.fr/session/1156).

## Avoiding common pitfalls with modern microservices testing - Eric Deandrea

Back to English to end the day, with a Quarkus session by Eric Deandrea from RedHat.

Since the first sentences I felt Eric was cool and master of the topic, which was confirmed after the talk and later during the conference when I had the chance to chat with him.

The main point of the talk being: it's very difficult to have all the services (or microservices) loaded from all the teams on a test environment.

If you think your microservices are small and decoupled, especially when you think that they are decoupled only because they are distributed. You have to test the lines connecting the dots, aka the contracts. Writing documentation on how to use an interface (a contract) is not the answer. The answer is testing and automation.

### Pact Contract Testing

It's the way of testing a contract on both sides. It's like unit testing across teams. For example the consumer could break a provider's test without the provider having done any change. Which means that the provider needs to know who the consumers are, which is not practical and cannot scale.

To test without the producers knowing who the clients are, you need to do a "Provider driven contract testing". Once you have defined your Swagger/OpenAPI definition for example, you can use that one to test on the client side so that any change on the client won't impact the producer.

Eric showed several examples in Quarkus (and oddly using Microsoft Edge on a Mac), but the main points are valid for any technology one may be use.

[Talk](https://rivieradev.fr/session/1147) by [<i class="fa-brands fa-twitter fa-sm" style="color: gray;"/>Eric Deandrea](https://twitter.com/edeandrea). [Slides here](https://speakerdeck.com/edeandrea/7-11-23-rivieradev-avoiding-common-pitfalls-with-modern-microservices-testing).

### Personal note

One day later I had the chance to talk to Eric. We were sitting at one of the tables were people were reviewing their talks or just do some tech exchange. I asked him about the main point of his talk, being that it's difficult for many services from several teams to be loaded reliably on a test environment, citing that in my experience having more than one reliable test systems with everything loaded had always been the norm.

He told me that it was the norm perhaps because of the small scale. I had to answer back with the actual scale I was talking about: not a handful of services, but hundreds (maybe more), sometimes written in different technology. It made me really appreciate the massive effort that is put to keep test environments up and running. 

## Notes

[^1]: If I could give a small advise to Aleš would be to variate a bit the tone of the voice.
[^2]: CNCF: Cloud Native Computing Foundation