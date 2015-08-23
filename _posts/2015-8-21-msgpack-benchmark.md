---
layout: post
title: MassagePack against the world of Python serializers
tags:
- python
- benchmark
---

<link rel="stylesheet" href="http://cdn.pydata.org/bokeh/release/bokeh-0.9.0.min.css" type="text/css" />

<script type="text/javascript" src="http://cdn.pydata.org/bokeh/release/bokeh-0.9.0.min.js"></script>

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

<script type="text/javascript">
    Bokeh.$(function() {
        var all_models = [{"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "tags": [], "doc": null, "dimension": 1, "ticker": {"type": "BasicTicker", "id": "c95ceae7-c045-4d1f-a9a6-fbaa88aed5d2"}, "id": "3fc12681-7856-43bf-8d0c-d9f5a778435d"}, "type": "Grid", "id": "3fc12681-7856-43bf-8d0c-d9f5a778435d"}, {"attributes": {"nonselection_glyph": null, "data_source": {"type": "ColumnDataSource", "id": "379e784a-b6ae-45d6-a830-9a19cec5d0b2"}, "tags": [], "doc": null, "selection_glyph": null, "id": "5cf89ab6-613f-457e-b6f5-f84c6ce688e4", "glyph": {"type": "Rect", "id": "dd2db34a-bc36-431f-9a15-b999ebc1ab53"}}, "type": "GlyphRenderer", "id": "5cf89ab6-613f-457e-b6f5-f84c6ce688e4"}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "dimensions": ["width", "height"], "tags": [], "doc": null, "id": "70571f94-8285-42ae-bcdc-bbf5cfbd9035"}, "type": "BoxZoomTool", "id": "70571f94-8285-42ae-bcdc-bbf5cfbd9035"}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "tags": [], "doc": null, "id": "c5eeae5a-ae67-4f20-8298-76b2f3b94932"}, "type": "PreviewSaveTool", "id": "c5eeae5a-ae67-4f20-8298-76b2f3b94932"}, {"attributes": {"line_color": {"value": "white"}, "fill_color": {"value": "#5ab738"}, "tags": [], "doc": null, "fill_alpha": {"value": 0.7}, "height": {"units": "data", "field": "json"}, "width": {"units": "data", "field": "width_cat"}, "y": {"field": "midjson"}, "x": {"field": "catjson"}, "id": "d4bfb34b-3cc3-4ec1-9405-80c978c3214d"}, "type": "Rect", "id": "d4bfb34b-3cc3-4ec1-9405-80c978c3214d"}, {"attributes": {"column_names": ["width_cat", "midujson", "cPickle", "catcPickle", "catjson", "stackedjson", "midcPickle", "ujson", "json", "stackedcPickle", "cat", "width", "zero", "catujson", "midmsgpack", "msgpack", "catmsgpack", "stackedmsgpack", "stackedujson", "midjson"], "tags": [], "doc": null, "selected": {"2d": {"indices": []}, "1d": {"indices": []}, "0d": {"indices": [], "flag": false}}, "callback": null, "data": {"width_cat": [0.2, 0.2], "midujson": [95.5, 960.0], "cPickle": [853, 8660], "catcPickle": ["10,000:0.2", "100,000:0.2"], "catjson": ["10,000:0.4", "100,000:0.4"], "stackedjson": [1453.0, 14710.0], "midcPickle": [426.5, 4330.0], "ujson": [191, 1920], "midjson": [600.0, 6050.0], "catujson": ["10,000:0.6", "100,000:0.6"], "json": [1200, 12100], "cat": ["10,000", "100,000"], "width": [0.8, 0.8], "stackedcPickle": [426.5, 4330.0], "stackedmsgpack": [2285.45, 23105.5], "midmsgpack": [41.45, 425.5], "msgpack": [82.9, 851.0], "catmsgpack": ["10,000:0.8", "100,000:0.8"], "zero": [2326.9, 23531.0], "stackedujson": [2148.5, 21720.0]}, "id": "379e784a-b6ae-45d6-a830-9a19cec5d0b2"}, "type": "ColumnDataSource", "id": "379e784a-b6ae-45d6-a830-9a19cec5d0b2"}, {"attributes": {"doc": null, "id": "0ea6c8b9-01d1-4588-b9ef-db28bca9b09f", "tags": []}, "type": "BasicTickFormatter", "id": "0ea6c8b9-01d1-4588-b9ef-db28bca9b09f"}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "tags": [], "doc": null, "id": "6963e0c5-6997-4313-beb4-6810a961f89e"}, "type": "HelpTool", "id": "6963e0c5-6997-4313-beb4-6810a961f89e"}, {"attributes": {"callback": null, "factors": ["10,000", "100,000"], "doc": null, "tags": [], "id": "83476b5a-4d07-4be0-81f9-faca040844dd"}, "type": "FactorRange", "id": "83476b5a-4d07-4be0-81f9-faca040844dd"}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "dimensions": ["width", "height"], "tags": [], "doc": null, "id": "84073145-3b0e-409d-9bca-4c63985b4c86"}, "type": "WheelZoomTool", "id": "84073145-3b0e-409d-9bca-4c63985b4c86"}, {"attributes": {"end": 13310.000000000002, "callback": null, "doc": null, "tags": [], "start": 0, "id": "dfc2dab2-ef2d-4aed-9c9a-04802cc76a0a"}, "type": "Range1d", "id": "dfc2dab2-ef2d-4aed-9c9a-04802cc76a0a"}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "orientation": "top_left", "tags": [], "doc": null, "label_text_font_size": {"value": "20pt"}, "id": "ad7bd8bc-602c-4c5d-8b04-efd4d05b1d6c", "legends": [["cPickle", [{"type": "GlyphRenderer", "id": "5cf89ab6-613f-457e-b6f5-f84c6ce688e4"}]], ["json", [{"type": "GlyphRenderer", "id": "07436c0f-da91-4034-ba98-a1ab388383d1"}]], ["ujson", [{"type": "GlyphRenderer", "id": "8e0b96d5-8a75-41bc-bc9d-7d5cb23f298b"}]], ["msgpack", [{"type": "GlyphRenderer", "id": "83b82bf0-ffa8-4680-a78e-b0dedc313a52"}]]]}, "type": "Legend", "id": "ad7bd8bc-602c-4c5d-8b04-efd4d05b1d6c"}, {"attributes": {"line_color": {"value": "white"}, "fill_color": {"value": "#f22c40"}, "tags": [], "doc": null, "fill_alpha": {"value": 0.7}, "height": {"units": "data", "field": "cPickle"}, "width": {"units": "data", "field": "width_cat"}, "y": {"field": "midcPickle"}, "x": {"field": "catcPickle"}, "id": "dd2db34a-bc36-431f-9a15-b999ebc1ab53"}, "type": "Rect", "id": "dd2db34a-bc36-431f-9a15-b999ebc1ab53"}, {"attributes": {"tags": [], "doc": null, "mantissas": [2, 5, 10], "id": "c95ceae7-c045-4d1f-a9a6-fbaa88aed5d2"}, "type": "BasicTicker", "id": "c95ceae7-c045-4d1f-a9a6-fbaa88aed5d2"}, {"attributes": {"nonselection_glyph": null, "data_source": {"type": "ColumnDataSource", "id": "379e784a-b6ae-45d6-a830-9a19cec5d0b2"}, "tags": [], "doc": null, "selection_glyph": null, "id": "07436c0f-da91-4034-ba98-a1ab388383d1", "glyph": {"type": "Rect", "id": "d4bfb34b-3cc3-4ec1-9405-80c978c3214d"}}, "type": "GlyphRenderer", "id": "07436c0f-da91-4034-ba98-a1ab388383d1"}, {"attributes": {"line_color": {"value": "white"}, "fill_color": {"value": "#407ee7"}, "tags": [], "doc": null, "fill_alpha": {"value": 0.7}, "height": {"units": "data", "field": "ujson"}, "width": {"units": "data", "field": "width_cat"}, "y": {"field": "midujson"}, "x": {"field": "catujson"}, "id": "34bf9356-7cc0-41c7-a5ab-388e0064f5ce"}, "type": "Rect", "id": "34bf9356-7cc0-41c7-a5ab-388e0064f5ce"}, {"attributes": {"nonselection_glyph": null, "data_source": {"type": "ColumnDataSource", "id": "379e784a-b6ae-45d6-a830-9a19cec5d0b2"}, "tags": [], "doc": null, "selection_glyph": null, "id": "8e0b96d5-8a75-41bc-bc9d-7d5cb23f298b", "glyph": {"type": "Rect", "id": "34bf9356-7cc0-41c7-a5ab-388e0064f5ce"}}, "type": "GlyphRenderer", "id": "8e0b96d5-8a75-41bc-bc9d-7d5cb23f298b"}, {"attributes": {"line_color": {"value": "white"}, "fill_color": {"value": "#df5320"}, "tags": [], "doc": null, "fill_alpha": {"value": 0.7}, "height": {"units": "data", "field": "msgpack"}, "width": {"units": "data", "field": "width_cat"}, "y": {"field": "midmsgpack"}, "x": {"field": "catmsgpack"}, "id": "a7c4786d-df71-4f7d-90bf-3bb6b09bf3cc"}, "type": "Rect", "id": "a7c4786d-df71-4f7d-90bf-3bb6b09bf3cc"}, {"attributes": {"nonselection_glyph": null, "data_source": {"type": "ColumnDataSource", "id": "379e784a-b6ae-45d6-a830-9a19cec5d0b2"}, "tags": [], "doc": null, "selection_glyph": null, "id": "83b82bf0-ffa8-4680-a78e-b0dedc313a52", "glyph": {"type": "Rect", "id": "a7c4786d-df71-4f7d-90bf-3bb6b09bf3cc"}}, "type": "GlyphRenderer", "id": "83b82bf0-ffa8-4680-a78e-b0dedc313a52"}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "dimensions": ["width", "height"], "tags": [], "doc": null, "id": "f35e42e9-88c0-48f5-a235-5fb5e39653a5"}, "type": "PanTool", "id": "f35e42e9-88c0-48f5-a235-5fb5e39653a5"}, {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d", "attributes": {"x_range": {"type": "FactorRange", "id": "83476b5a-4d07-4be0-81f9-faca040844dd"}, "right": [], "above": [], "tags": [], "tools": [{"type": "PanTool", "id": "f35e42e9-88c0-48f5-a235-5fb5e39653a5"}, {"type": "WheelZoomTool", "id": "84073145-3b0e-409d-9bca-4c63985b4c86"}, {"type": "BoxZoomTool", "id": "70571f94-8285-42ae-bcdc-bbf5cfbd9035"}, {"type": "PreviewSaveTool", "id": "c5eeae5a-ae67-4f20-8298-76b2f3b94932"}, {"type": "ResizeTool", "id": "ba150d8c-b50a-44f2-8bc8-93ea44e06949"}, {"type": "ResetTool", "id": "1aaabad2-8d15-4466-b902-8a016654bfa0"}, {"type": "HelpTool", "id": "6963e0c5-6997-4313-beb4-6810a961f89e"}], "title": "Dumps operation time", "renderers": [{"type": "CategoricalAxis", "id": "64a17d90-f656-4f2d-a903-f4d4dbe8749b"}, {"type": "LinearAxis", "id": "85491035-da09-429e-96e3-055ec0006d11"}, {"type": "Grid", "id": "3fc12681-7856-43bf-8d0c-d9f5a778435d"}, {"type": "GlyphRenderer", "id": "5cf89ab6-613f-457e-b6f5-f84c6ce688e4"}, {"type": "GlyphRenderer", "id": "07436c0f-da91-4034-ba98-a1ab388383d1"}, {"type": "GlyphRenderer", "id": "8e0b96d5-8a75-41bc-bc9d-7d5cb23f298b"}, {"type": "GlyphRenderer", "id": "83b82bf0-ffa8-4680-a78e-b0dedc313a52"}, {"type": "Legend", "id": "ad7bd8bc-602c-4c5d-8b04-efd4d05b1d6c"}], "plot_width": 600, "extra_y_ranges": {}, "extra_x_ranges": {}, "tool_events": {"type": "ToolEvents", "id": "7b8b3ec1-f831-4253-8122-9a441684b418"}, "plot_height": 400, "doc": null, "id": "c9357378-031f-42ca-9645-3dd61291da0d", "y_range": {"type": "Range1d", "id": "dfc2dab2-ef2d-4aed-9c9a-04802cc76a0a"}, "below": [{"type": "CategoricalAxis", "id": "64a17d90-f656-4f2d-a903-f4d4dbe8749b"}], "left": [{"type": "LinearAxis", "id": "85491035-da09-429e-96e3-055ec0006d11"}]}}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "tags": [], "ticker": {"type": "CategoricalTicker", "id": "e4860faf-3a46-4bc0-95bb-ebac837f0372"}, "major_label_text_baseline": "hanging", "doc": null, "major_label_orientation": 0, "axis_label": "Number of elements", "formatter": {"type": "CategoricalTickFormatter", "id": "f8f2fc3a-6f67-4a47-a5ea-6726c515e0dc"}, "major_label_text_font_size": {"value": "20pt"}, "id": "64a17d90-f656-4f2d-a903-f4d4dbe8749b"}, "type": "CategoricalAxis", "id": "64a17d90-f656-4f2d-a903-f4d4dbe8749b"}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "tags": [], "doc": null, "id": "ba150d8c-b50a-44f2-8bc8-93ea44e06949"}, "type": "ResizeTool", "id": "ba150d8c-b50a-44f2-8bc8-93ea44e06949"}, {"attributes": {"geometries": [], "tags": [], "doc": null, "id": "7b8b3ec1-f831-4253-8122-9a441684b418"}, "type": "ToolEvents", "id": "7b8b3ec1-f831-4253-8122-9a441684b418"}, {"attributes": {"doc": null, "id": "e4860faf-3a46-4bc0-95bb-ebac837f0372", "tags": []}, "type": "CategoricalTicker", "id": "e4860faf-3a46-4bc0-95bb-ebac837f0372"}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "tags": [], "ticker": {"type": "BasicTicker", "id": "c95ceae7-c045-4d1f-a9a6-fbaa88aed5d2"}, "doc": null, "axis_label": "Time (ms)", "formatter": {"type": "BasicTickFormatter", "id": "0ea6c8b9-01d1-4588-b9ef-db28bca9b09f"}, "major_label_text_font_size": {"value": "20pt"}, "id": "85491035-da09-429e-96e3-055ec0006d11"}, "type": "LinearAxis", "id": "85491035-da09-429e-96e3-055ec0006d11"}, {"attributes": {"doc": null, "id": "f8f2fc3a-6f67-4a47-a5ea-6726c515e0dc", "tags": []}, "type": "CategoricalTickFormatter", "id": "f8f2fc3a-6f67-4a47-a5ea-6726c515e0dc"}, {"attributes": {"plot": {"subtype": "Chart", "type": "Plot", "id": "c9357378-031f-42ca-9645-3dd61291da0d"}, "tags": [], "doc": null, "id": "1aaabad2-8d15-4466-b902-8a016654bfa0"}, "type": "ResetTool", "id": "1aaabad2-8d15-4466-b902-8a016654bfa0"}];
        Bokeh.load_models(all_models);
        var plots = [{'modeltype': 'Plot', 'elementid': '#3deb6733-b07f-4dcc-bc01-0bd1e12513dd', 'modelid': 'c9357378-031f-42ca-9645-3dd61291da0d'}];
        for (idx in plots) {
            var plot = plots[idx];
            var model = Bokeh.Collections(plot.modeltype).get(plot.modelid);
            Bokeh.logger.info('Realizing plot:')
            Bokeh.logger.info(' - modeltype: ' + plot.modeltype);
            Bokeh.logger.info(' - modelid: ' + plot.modelid);
            Bokeh.logger.info(' - elementid: ' + plot.elementid);
            var view = new model.default_view({
                model: model,
                el: plot.elementid
            });
            Bokeh.index[plot.modelid] = view;
        }
    });
</script>

<div class="plotdiv" id="3deb6733-b07f-4dcc-bc01-0bd1e12513dd"></div>

The plot only shows data with the size of list: `10,000`, and `100,000`. The performance of these libraries for small list is so fast so that plotting them together with the big list data will not make sense.

The performance of `msgpack` is an order of magnitude against `cPickle` & `json`; and at least 2 times faster then `ujson`. The performance is pretty consistent for various size of the test list.

# Loads function performance

| Size of list | cPickle |  json | ujson | msgpack |
|:------------:|:-------:|:-----:|:-----:|:-------:|
|      10      |  0.712  | 0.604 | 0.179 | 0.073   |
|      100     |   7.25  |  6.46 | 2.37  | 0.82    |
|     1,000    |   73.7  |  67.5 | 27.2  | 11.1    |
|    10,000    |   764   |  699  | 287   | 115     |
|    100,000   |   7710  |  7130 | 2840  | 1190    |

![Loads operation time plot]({{ site.url }}/images/plot/2015-08-21-msgpack-benchmark-plot-dumps.png)

The performance of `msgpack` is still an order of magnitude against `cPickle` & `json`; and at least 2 times faster then `ujson`. The time it takes for deserializing is a bit longer than serializing for `msgpack` and `ujson`. On the contrary, `cPickle` and `json` deserialize faster than serialize.

# Size of the serialized message

The size of the serialized message from these libraries is measured in *KB*

| Size of list | cPickle |   json  |  ujson  | msgpack |
|:------------:|:-------:|:-------:|:-------:|:-------:|
|      10      |    32   |    32   | 28      | 16      |
|      100     |   300   |   320   | 244     | 140     |
|     1,000    |  2,968  |  3,168  | 2,404   | 1,372   |
|    10,000    |  29,704 |  31,676 | 24,016  | 13,692  |
|    100,000   | 297,312 | 316,744 | 240,132 | 136,892 |

![Size of message]({{ site.url }}/images/plot/2015-08-21-msgpack-benchmark-plot-dumps.png)

`json` and `ujson` produces **JSON** which is human-readable compared to `cPickle` and `msgpack`. However, the size of the message produced is sometimes important. That's where `msgpack` shines. `msgpack` can save **40%** the size of messages produced when serialized to file.




