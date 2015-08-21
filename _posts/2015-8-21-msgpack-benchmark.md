---
layout: post
title: Msgpack against the world of Python serializers
tags: Python, benchmark
---

> MessagePack is an efficient binary serialization format.
> It lets you exchange data among multiple languages like JSON.
> But it's ***faster*** and ***smaller***.

I am involved in multiple projects where I need to serialize data and send to a message queue. I found **Python pickle** is very useful and included with Python. However there are multiple ways to exchange data, **JSON** is unarguably one of the most common format, and **MessagePack** is a new guy in town. In order to choose a serializer, speed and the size of the message are important. And we will see in this post how all of them performs compared to each other, especially **MessagePack**.

# Setup

I will test [Python msgpack 0.4.6](https://pypi.python.org/pypi/msgpack-python/) against the default [pickle](https://docs.python.org/2/library/pickle.html#module-cPickle) format of Python with **cPickle 1.71**, the [default Python json (2.0.9) package](https://docs.python.org/2/library/json.html), and a highly optimized, written mostly in C [ultrajson 1.33](https://pypi.python.org/pypi/ujson) package.

Operations: **dumps** and **loads** will be tested. The runtime is measured in **ms**.

The environment for testing is `Python 2.7.10`, with Intel(R) Core(TM) i3-3240 CPU @ 3.40GHz processor. The data point used for this benchmark is created from a dictionary of various data type. Then I generate different lists of different sizes of: `10`, `100`, `1,000`, `10,000`, and `100,000`.

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
data_10 = [generate_small_data() for each in xrange(10)]
data_100 = [generate_small_data() for each in xrange(100)]
data_1000 = [generate_small_data() for each in xrange(1000)]
data_10000 = [generate_small_data() for each in xrange(10000)]
data_100000 = [generate_small_data() for each in xrange(100000)]
```

# Dumps function performance

| Size of list | cPickle |  json | ujson | msgpack |
|:------------:|:-------:|:-----:|:-----:|:-------:|
|      10      |  0.823  | 1.080 | 0.175 | 0.0714  |
|      100     |   8.34  |  11.9 | 1.81  | 0.792   |
|     1,000    |    85   |  120  | 18.7  | 8.15    |
|    10,000    |   853   |  1200 | 191   | 82.9    |
|    100,000   |   8660  | 12100 | 1920  | 851     |

# Loads function performance

| Size of list | cPickle |  json | ujson | msgpack |
|:------------:|:-------:|:-----:|:-----:|:-------:|
|      10      |  0.712  | 0.604 | 0.179 | 0.073   |
|      100     |   7.25  |  6.46 | 2.37  | 0.82    |
|     1,000    |   73.7  |  67.5 | 27.2  | 11.1    |
|    10,000    |   764   |  699  | 287   | 115     |
|    100,000   |   7710  |  7130 | 2840  | 1190    |

# Size of the serialized message

The size of the serialized message from these libraries is measured in *KB*

| Size of list | cPickle |   json  |  ujson  | msgpack |
|:------------:|:-------:|:-------:|:-------:|:-------:|
|      10      |    32   |    32   | 28      | 16      |
|      100     |   300   |   320   | 244     | 140     |
|     1,000    |  2,968  |  3,168  | 2,404   | 1,372   |
|    10,000    |  29,704 |  31,676 | 24,016  | 13,692  |
|    100,000   | 297,312 | 316,744 | 240,132 | 1190    |
