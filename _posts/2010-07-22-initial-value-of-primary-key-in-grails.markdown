---
date: '2010-07-22 21:00:25'
layout: post
slug: initial-value-of-primary-key-in-grails
status: publish
title: Initial value of primary key in Grails
wordpress_id: '4'
categories:
- Grails
- Groovy
- Programming
tags:
- english
- grails
- groovy
- programming
---

I’ve recently came across a small issue in my Grails application. I had to set an initial value of auto-generated identifiers of my objects. Moreover, I had to use two different values in two classes. For example, identifiers in one table in database should start from 50, while in another table – from 1000. How to get this in Grails domain classes?




You have to use SequenceStyleGenerator with its parameter – initial_value. You can do this with generator phrase in mapping clousure:





    class FirstClass {
      static mapping = {
        id(generator: 'org.hibernate.id.enhanced.SequenceStyleGenerator',
           params: [initial_value: 50])
      }
      ...
    }

    class SecondClass {
      static mapping = {
        id(generator: 'org.hibernate.id.enhanced.SequenceStyleGenerator',
           params: [initial_value: 1000])
      }
      ...
    }




Unfortunately, in this case you’ll finish with the same sequence generator in both classes. That means that you’ll get one sequence shared between two tables. How to avoid this? Just specify sequence name using next parameter – sequence_name:




    class FirstClass {
      static mapping = {
        id(generator: 'org.hibernate.id.enhanced.SequenceStyleGenerator',
           params: [sequence_name: 'first_seq', initial_value: 50])
      }
      ...
    }

    class SecondClass {
      static mapping = {
        id(generator: 'org.hibernate.id.enhanced.SequenceStyleGenerator',
           params: [sequence_name: 'second_seq', initial_value: 1000])
      }
      ...
    }




What do you think about my solution? Do you know another one?




This article was previously posted at [Groovy Zone](http://groovy.dzone.com/tips/initial-value-primary-key).
