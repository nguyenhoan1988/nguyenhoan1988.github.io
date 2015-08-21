---
layout: post
title: Msgpack against the world of Python serializers
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

<table>
  <tr>
    <th>Size of list</th>
    <th>cPickle</th>
    <th>json</th>
    <th>ujson</th>
    <th>msgpack</th>
  </tr>
  <tr>
    <td>10</td>
    <td>0.823</td>
    <td>1.080</td>
    <td>0.175</td>
    <td>0.0714</td>
  </tr>
  <tr>
    <td>100</td>
    <td>8.34</td>
    <td>11.9</td>
    <td>1.81</td>
    <td>0.792</td>
  </tr>
  <tr>
    <td>1,000</td>
    <td>85</td>
    <td>120</td>
    <td>18.7</td>
    <td>8.15</td>
  </tr>
  <tr>
    <td>10,000</td>
    <td>853</td>
    <td>1200</td>
    <td>191</td>
    <td>82.9</td>
  </tr>
  <tr>
    <td>100,000</td>
    <td>8660</td>
    <td>12100</td>
    <td>1920</td>
    <td>851</td>
  </tr>
</table>
