---
title: Elasticsearch Java API自定义排序
cdn: header-off
date: 2018-10-16 11:18:20
header-img: "/img/es.png"
tags:
    - Elasticsearch
    - Java
---
## 场景
查询ES中时，在数据量大、数据时效性强的情况下，要查询到针对性强的少量数据，往往过滤条件会有很多，这个时候我们通常希望通过自己提供的关键字列表和条件对数据进行过滤。

在音乐项目的新闻模块中，新闻的下发要求为：三天内发布的新闻，按照国家条件过滤后下发排序为：榜单歌手热度 > 新闻发布时间。