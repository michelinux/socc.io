---
layout: post
title: Riviera DEV 2023 Trip Report, Part Two
date: 2023-08-26 10:25 +0200
published: true
---

Riviera DEV 2023 was my first Dev conference. It's held in the French Riviera, it has a great atmosphere, exceptional food and provides no excuses for you not to attend. For all those reasons, and secondary because I was one of the speakers and I am leaving at a few bus stops, I attended day 2 and 3 of the conference. This is my **Riviera DEV 2023 Trip Report, Part Two**. There is also a [Part One]({% post_url 2023-08-14-riviera-dev-2023-trip-report-part-one %}).

I did attend less talks than the [day before]({% post_url 2023-08-14-riviera-dev-2023-trip-report-part-one %}). I still had to rehearsal my own talk, and had several corridors chats.

## Women's journey in tech - Pooneh Mokari

The second keynote touched me deeply. Because of the personal history of Pooneh Mokari, who is from Iran. Because of the numbers that were presented. And because of some aspects of the situation of women in tech that I was not aware of. Let's go in order.

Pooneh studied her master in France, and she recalls that in a group project she was the only girl. All the boys didn't even talk to her. She assumes this was because of her gender[^1].

We already know that diversity is good for better society, and also good for a higher productivity. However the reality is bad. In tech we only have 22% of women, while overall in the world the women workforce is 45%-50% (not counting India).

There are several aspects that I never considered.

#### Lack of role model

- You cannot become someone who don't exist
- There are great names of are women, but they are not close to us

This reminds me the first days of university. In a class of 250 people starting software engineering there were only a handful of women. Nothing prevented them to sign up. Now I understand.

#### Positive Discrimination

_You will find job easier because you are women and companies need to have quota._ This is something that will make someone feel less: am I hired or promoted only because I am a woman?

#### Enjoy and Share

We need to change stereotypes using social media and podcasts. In other words: not everybody in a tech company should love video-games.

I don't love video-games.

#### Job interview

The interviewers are often mostly men and white. The candidates from minorities may not feel they fit, and this will influence the interview.

To sum up, there is still a long way. If we need more women in tech, adjusting our interviews or companies culture won't be enough. We need more girls choosing to study engineering and STEM in general. We need more role models.

Pooneh works at Criteo. Her talk was the [keynote of the third day of Riviera DEV 2023](https://rivieradev.fr/program). She also wrote an article on public speaking that you can find on Medium[^2].

The video of her talk is not yet published by Riviera DEV, but [she has delivered it](https://www.youtube.com/watch?v=6431ft_P1rQ) at other conferences.


## Test Containers - Oleg Šelajev 

![Test Containers](https://docs.localstack.cloud/user-guide/integrations/testcontainers/testcontainers-logo.svg){: .right w="300"}

Test Containers are throw away instances of databases and other services that you may need to run your tests. So you basically just add some code like:

```java
final mongoDBContainer = new MongoDBContainer(DockerImageName.parse("mongo:6.0.8"));
mongoDBContainer.start();
```

Then you can use Test Containers API to get the address to connect. It all seems very convenient and easy, mostly also because Oleg is from AtomicJars which is the company behind Test Containers.

Said so, the approach is similar to what Quarkus already offers natively, for what I saw. I come from a world where this kind of approach is not available and you have to fight with scripts and pray that the externally deployed instances for DB, Kafka, etc. always work (they always do thanks to the amazing people in charge of those test instances). This looks almost like magic, or my world looks almost like the dark ages.

With some more reflections, I have a few doubts about Test Containers:

- There are many modules, but each module has to be adapted for each language. So not all modules may be available for each language. 
- Test Containers is not available at all for C++ (it is for Rust, it can't be that difficult!).
- In term of resources and energy, I am concerned about the waste of having one DB instance for each user.
- Quarkus already has this kind of technology.

Maybe I should ask [<i class="fa-brands fa-twitter fa-sm" style="color: gray;"/>Oleg Šelajev](https://twitter.com/shelajev) about my doubts. In the while his slides should be [published soon](https://rivieradev.fr/session/1119).

## Dropping 25TB in 3 years: the tale of a Relational Database on diet, for safety and for cost. 

[My talk](https://rivieradev.fr/session/1120), I won't say more.

## Beyond the talks

There is a lot more than talks to listen to at a conference. There was food, there was also ice-cream. There was also a VR reconstruction of Notre Dame. 

I had chance to talk to people at boots, or just other speakers at the tables were we were reviewing our slides or just doing some work. I had some very good time with Danica and Oleg, and an interesting chat with Eric Deandrea about Test Environments. And I could stop by at the RedHat boot to just ask some Quarkus questions, get some answers and spark some discussions (I should review what I was told about feature toggles). Despite the very French imprint of the conference, I was still able to talk to people from all over the world, in four languages (nobody from Hong Kong <i class="fas fa-laugh fa-sm" style="color: gray;"/>). And got many stickers, with the only regret of not getting more for my future laptops.

Did I mention there was also <i class="fas fa-ice-cream fa-sm" style="color: gray;"/> ice cream?


## Notes

[^1]: My objection (the only one to her compelling talk) is that in my experience French people just won't pay the effort to speak English _parce que ça lui fait chier_. I have been lucky enough to have friends that were patient with me when I couldn't speak French well yet but after learning some better French I observed my interlocutors felt less annoyed.

[^2]: Which I won't link because Medium is kind-of paywalled, cumbersome to use, Google is not indexing it well and their search doesn't work. In other words: I couldn't find it.