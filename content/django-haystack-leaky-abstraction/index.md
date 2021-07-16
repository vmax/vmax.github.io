---
title: Boolean fields in django-haystack
date: 2021-07-16
description: are a leaky abstraction
---

Say, you're using [django-haystack](https://pypi.org/project/django-haystack/) to integrate search & faceting
to your project, backed by ElasticSearch. The index that you're building has a boolean field:

```
class SampleIndex(indexes.SearchIndex, indexes.Indexable):
    text = indexes.CharField(document=True)
    sample_bool = indexes.BooleanField(model_attr='sample_bool')
```

Then, you try filtering by that boolean field like you'd normally do with a Django's `QuerySet`:

```
>>> from haystack.query import SearchQuerySet
>>> SearchQuerySet().count()
42
>>> SearchQuerySet().filter(sample_bool=True).count()
42
```

Is it _really_ correct? Given that there are only boolean values in existence are `True` and `False`,
you'd expect filtering on `False` should give you exactly zero results. Let's verify that:

```
>>> SearchQuerySet().filter(sample_bool=False).count()
42
```

![wat](https://user-images.githubusercontent.com/499267/125994957-d74f0f75-0ccd-4e9d-90a5-835fe6f75b4b.jpg)


Well, that's because when translating a query to language that ElasticSearch understands, both `True` and `False`
are turned into their stringified representations (`"True"` and `"False"`) which both evaluate to a truthy value. In fact,
any non-empty string would also yield the same result:

```
>>> SearchQuerySet().filter(sample_bool='example').count()
42
```

To get the results we expect we need to pass `"true"` and `"false"` (lowercase strings):

```
>>> SearchQuerySet().filter(sample_bool='true').count()
42
>>> SearchQuerySet().filter(sample_bool='false').count()
0
```

ElasticSearch 5 warns you against passing boolean values directly:

```
[WARN ][o.e.d.i.m.BooleanFieldMapper] searching using boolean value [False] is deprecated, please use [true] or [false]
```

And that's what is called a [leaky abstraction](https://en.wikipedia.org/wiki/Leaky_abstraction). In order to query on
boolean fields, users need to be aware of how the underlying implementation behaves.
