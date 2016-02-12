---
date: 2012-11-11 15:21:23 +0100
layout: post
slug: schema-design-in-mongodb
status: publish
title: Schema Design in MongoDB
categories:
- Courses
- Databases
- MongoDB
---

Recently, I started MongoDB for Developers course from [10gen Education](https://education.10gen.com). The third week brought some very interesting information on schema design.

I decided to write it down here for further reference but I hope it will also be useful for others.

## MongoDB limitations

MongoDB **do not provide foreign key constraints** known from relational databases. How can we deal with it? Well, one option is **embedding**. If you don't separate documents into a few collections but embed one into other, you basically avoid using foreign key at all! It of course isn't a silver bullet - it depends on your data usage pattern or other MongoDB limitations (like max. 16MB per document), but it's definitely worth considering.

Moreover, Mongo **don't provide transactions**. It looks quite serious but it actually can be worked around easily, because of a few solutions.

First of all, MongoDB supports **atomic operations**. It means you have a guarantee that a single document will be persisted atomically, ie. saved all or nothing. How does it help the situation? Well, if you embed a document into another, you don't need to worry about transactions at all. You save one document (atomically!), instead of a few, which would need transactional approach.

If you can't or don't want to embed documents, you still have some options. First of all, you can implement some sort of **locking on the application level**. For example, it can be critical sections, semaphores, etc.

The last option is to **tolarate** a little bit of inconsistency. Sometimes, actually more often nowadays, it isn't crucial to provide the same data to all users at the same time. If you consider social networks, it makes sense to show the fact that you made a new friendship a bit later to some of your friends, than to others. In this case the goal is to achieve *eventual consistency*, not the immediate consistency.

## Relations

In MongoDB, you can achieve relations between documents on one of four ways:

 * put a foreign key in one of these documents
 * put foreign keys in both documents
 * embed one document into another

Which one to choose depends on a few factors, including relation type.

### One to One

In one to one relation you can ask yourself three questions.

**How often will I use the data?** If the data won't be used often as a single piece, you probably should separate it into two documents.

**How big is the data?** If the data size exceeds 16 MB then it's a no brainer - you need to separate it. Also, when the data is big, however under 16 MB, but you constantly update only a part of it, it probably makes sense to separate it as well to avoid update overhead. Of course, it's also important here to think about how the data will grow up in the future.

**Do I need to update the data atomically?** If the answer is yes and there are no other contraindications, then it makes sense to keep it together as mentioned above.

### One to Many

The question here is: is it real one to many or maybe it's rather one to *few*? So, if you have a people-city relation, it makes sense to use *true linking* and two collections. However, if you have a blog post - comments relation, then you probably should decide to embedding.

### Many to Many

As a rule of thumb, it makes the most sense to create two collections and link them using array fields. If you'd like to make this connection bidirectional, then the array should be in both collections. It becomes more powerful together with [multikey indexes](http://www.mongodb.org/display/DOCS/Multikeys).

## Conclusion

To sum up, we can say that a rule of thumb is to embed. Beside the advantages mentioned above, there is one more - performance. Embedded documents lays near each other on the HDD, probably in the same sector, so reading them is much faster because of hard drives nature.

However, it all depends on the data access patterns of your application. You need to consider potential duplication anomalies as well as performance. I hope the above recommendations will help with the decission.
