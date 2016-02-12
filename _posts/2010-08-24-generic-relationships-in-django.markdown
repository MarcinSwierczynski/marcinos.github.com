---
date: 2010-08-24 17:53:28 +0100
layout: post
slug: generic-relationships-in-django
status: publish
title: Generic relationships in Django
wordpress_id: '28'
categories:
- Django
- Programming
- Python
tags:
- django
- english
- programming
- python
---

Last week I've completed an interesting task within our project: building internal link-shortening system based on persisted objects, not on constant URLs. The main requirement was to get as loosely-coupled design as possible. In the perfect world the "shortenable" classes should know absolutely nothing about link-shortening mechanism. How to get such a flexibility in Django? I've used the [ContentTypes framework](http://docs.djangoproject.com/en/1.2/ref/contrib/contenttypes/) with [generic relations](http://docs.djangoproject.com/en/1.2/ref/contrib/contenttypes/#generic-relations) and it works pretty well!

First of all, we need to prepare the model class which will keep references to every "shortanable" object and its shortening key which will be used in URLs. The key-related part is quite easy, but what about the former requirement? How it is possible to keep a relation to an object of type we don't even know? The answer is: [ContentTypes framework](http://docs.djangoproject.com/en/1.2/ref/contrib/contenttypes/). It gives us an ability to get an identifier of every class within Django-based application. So, we've got all we need now. Just take a look into a small code snippet to see this feature in action:

```python
    class ShorteningKey(models.Model):
        """ Contains a key for a generic object used in short link resolving """
        key = models.SlugField(unique=True)
        content_type = models.ForeignKey(ContentType)
        object_id = models.PositiveIntegerField()
        content_object = generic.GenericForeignKey()
```


Quite neat, isn't it? Now it is possible to get a shortening key for a given object. We'll do that with help of the following method.

```python
    @classmethod
    def get_shortening_key_for_instance(cls, instance):
        """ Return ShorteningKey object associated with given model object.
            May throw DoesNotExist exception """
        type = ContentType.objects.get_for_model(instance)
        return ShorteningKey.objects.get(content_type__pk=type.id, object_id=instance.id)
```


The last thing to do is to generate a unique key for every object. As I've mentioned, it was extremely important to keep "shortenable" classes as clean as possible, so hard-coding shortening key generating method within the scope of each relevant class wasn't a good idea. Instead of this, I've used [Signals](http://docs.djangoproject.com/en/1.2/topics/signals/). Now, we have to write key-generation method and associate it with `post_save` signal. Why do we use the `post_save`? Because we need persisted instance to use it within ContentTypes framework. The described function is listed below while process of attaching it to the signal will be shown later.

```python
    def generate_shortening_key(sender, **kwargs):
        """ Generates shortening key for given object """
        instance = kwargs['instance']
        new_instance_created = kwargs['created']

        if new_instance_created:
            attempt = 0
            while attempt < 100:
                try:
                    shortening_key = ShorteningKey(content_object=instance, key=generate_key())
                    shortening_key.save()
                    break
                except DatabaseError:
                    attempt += 1
            else:
                logging.warning("Cannot create link shortening for " + str(instance))
```


We'll also need a function which will delete unnecessary key after object deletion. It's quite simple and similar so I won't put its code here.

The whole thing is almost ready, we just need to provide a function which is responsible for retrieving shortened URL.


```python
    def get_shortened_url(instance):
        """ Returns shortened url to given instance. If there is no shortened url,
            it tries to return full url to object. If there is no one, it returns empty string.
            This function is dynamically attached to any class with link shortening enabled """
        try:
            shortening_key = ShorteningKey.get_shortening_key_for_instance(instance)
            return shortening_key.get_shortened_url()
        except ShorteningKey.DoesNotExist:
            try:
                return instance.get_absolute_url()
            except (TypeError, AttributeError):
                return ''
```


It will be perfect if the function would be a member of each "shortenable" class, won't it? We will attach it dynamically to each of them in the same loop which is responsible for signals attaching.

```python
    LINK_SHORTENING_ENABLED_CLASSES = (Class1,Class2,...)
    for cls in LINK_SHORTENING_ENABLED_CLASSES:
        post_save.connect(generate_shortening_key, sender=cls)
        post_delete.connect(delete_shortening_key, sender=cls)
        cls.get_shortened_url = get_shortened_url
```


That's all! We've got fully featured link-shortening system with loosely-coupled classes. You can easily adjust this example and make it appropriate for your particular needs. For example, you can build a [comment system](http://docs.djangoproject.com/en/1.2/ref/contrib/comments/), just like Django team did :)
