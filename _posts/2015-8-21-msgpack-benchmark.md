---
layout: post
title: Msgpack against the world of Python serializers
---

> MessagePack is an efficient binary serialization format.
> It lets you exchange data among multiple languages like JSON.
> But it's ***faster*** and ***smaller***.

There are multiple ways to exchange data, **JSON** is unarguably one of the most common format. **MessagePack** is a new guy in town. And we will see in this post how this guy performs compared to others.

I will test [Python msgpack](https://pypi.python.org/pypi/msgpack-python/) against the default [pickle](https://docs.python.org/2/library/pickle.html#module-cPickle) format of Python with **cPickle**, the [default Python json package](https://docs.python.org/2/library/json.html), and a highly optimized, written mostly in C [ultrajson](https://pypi.python.org/pypi/ujson) package.

The data point used for this benchmark is created from a dictionary of various data type. Then I generate different lists of different sizes of: `10,000`, `50,000`, `100,000`, `500,000`, and `1,000,000`.

``` Python
from random import random, randint

def generate_small_data():
    return {
        'string': """
            Lorem ipsum dolor sit amet, consectetur adipiscing
            elit. Mauris adipiscing adipiscing placerat.
            Vestibulum augue augue,
            pellentesque quis sollicitudin id, adipiscing.
            """,
        'list': range(100),
        'dict': {i: i * random() for i in xrange(100)},
        'int': randint(-10 ** 10, 10 ** 10),
        'float': randint(-10 ** 10, 10 ** 10) * random()
    }

small_data = generate_small_data()
data_10000 = [generate_small_data() for each in xrange(10000)]
data_50000 = [generate_small_data() for each in xrange(50000)]
data_100000 = [generate_small_data() for each in xrange(100000)]
data_500000 = [generate_small_data() for each in xrange(500000)]
data_1000000 = [generate_small_data() for each in xrange(1000000)]
```
