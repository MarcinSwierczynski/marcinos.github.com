---
date: 2023-03-27 14:23:00 +0100
layout: post
slug: alexa-skill-with-quarkus-native
status: publish
title: Alexa Skill with Quarkus Native
categories:
- Programming
- Java
- Quarkus
- Quarkus Native
---

## Airly Alexa Skill

Some time ago I wrote a skill for Alexa fetching current air pollution data from Airly - the air pollution service popular in Poland.

The skill is very simple - one says "Alexa, open air quality" and it replies something like "Great air here today! The Common Air Quality Index is 12".

I wrote it in Java - event it's not the best choice for AWS Lambda - because why not? :)

The code is available at https://github.com/MarcinSwierczynski/alexa-airly 

## The performance

I implemented the above with standard approach - using Java Corretto and Alexa Skills Kit SDK for Java.

It wasn't a surprise, but the Lambda calls duration were pretty long, especially because of cold start.

![Alexa Skills Kit SDK for Java](/img/posts/alexa/aws-sdk.png)

As you can see, it took 8-9 s for a single call.

## Quarkus to the rescue?

I decided to use Quarkus to improve the situation.

The code is at https://github.com/MarcinSwierczynski/alexa-airly-quarkus

The important parts are:
- `QuarkusDelegateStreamLambda` is needed even it does nothing
- it is referred in `application.properties` as `quarkus.lambda.handler=alexa`
- `slf4j` should be excluded from `ask-sdk` artifact
- Quarkus needs to be initialized e.g. in a static block in a subclass of `SkillStreamHandler`

![Quarkus](/img/posts/alexa/quarkus.png)

It worked well with the calls duration ~12 s for a cold start and ~4 s later on.

Better, but so good given the nature of the air pollution skill - you usually use it a few times a day.

## Quarkus Native

So, the natural remedy was Quarkus Native.

The code is at https://github.com/MarcinSwierczynski/alexa-airly-quarkus-native

The important parts are:
- `io.quarkiverse.amazonalexa:quarkus-amazon-alexa` dependency
- `QuarkusDelegateStreamLambda` now does JSON/bytes conversion
- it is referred in `application.properties` as `quarkus.lambda.handler=alexa`
- `slf4j` should still be excluded from `ask-sdk` artifact
- Quarkus does _not_ need to be initialized in a subclass of `SkillStreamHandler`

![Quarkus Native](/img/posts/alexa/quarkus-native.png)

The calls are around 800 ms, so an order of magnitude less then whit standard approach!

## Conclusion
Quarkus Native seems to be a great alternative for writing Lambdas. Especially if you have Java competences in your team and don't want to introduce Go/Python.
