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

## 实现
### 数据结构
``` json
{
  "took": 94,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 51,
    "max_score": 1,
    "hits": [
      {
        "_index": "data_music_news",
        "_type": "music_data",
        "_id": "127591",
        "_score": 1,
        "_source": {
          "id": 127591,
          "news_id": "74ec53fd809728e91ec159f2a154b309",
          "title": "Halsey Dishes on B-Day Italy Trip With G-Eazy at 2018 AMAs",
          "thumbnail": {
            "height": 315,
            "url": "https://eonlinethumbs-a.akamaihd.net/images/323/651/lfrc_2018amas_halsey_276905_560x315_1340545091592.jpg",
            "width": 560
          },
          "published_at": 1539114900000,
          "url": "https://www.eonline.com/videos/276905/halsey-dishes-on-b-day-italy-trip-with-g-eazy-at-2018-amas",
          "source": "E! News",
          "countries": [
            "US"
          ],
          "artists": [
            "G-Eazy",
            "Halsey",
            "Eazy"
          ],
          "create_time": 1539144039000,
          "update_time": 1539144039000,
          "activated": true
        }
      }
    ]
  }
}
```
### Java API
#### 原始版本
大致思路为，从ES中查询出符合时间条件并且新闻的歌手关键词包含热门歌手的新闻，再对查询结果按照歌手Retain。
``` java
    public List<ENews> queryByTopArtists(List<String> artists, Collection<String> excludeIds, Integer size) {
        List<ENews> eNewsList = new ArrayList<>();

        // 查询5倍的数据供后期歌手标签去重
        int querySize = size * 5;

        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();

        if (CollectionUtils.isNotEmpty(excludeIds)) {
            boolQueryBuilder.mustNot(QueryBuilders.termsQuery("news_id", excludeIds));
        }
        boolQueryBuilder.must(QueryBuilders.termQuery("activated", true));

        Map<String, Object> params = new HashMap<>();

        params.put("artists", artists);

        StringBuilder scriptSortCode = new StringBuilder();

        scriptSortCode.append("if (new Date().getTime() - doc['published_at'].date.getMillis() < 86400000 * 12 && doc['artists'] != null) {");
        scriptSortCode.append("    for (int i = 0; i < params.artists.length; ++i) {");
        scriptSortCode.append("        if (doc['artists'].contains(params.artists[i])) {");
        scriptSortCode.append("             return i;");
        scriptSortCode.append("        }");
        scriptSortCode.append("    }");
        scriptSortCode.append("    return params.artists.length;");
        scriptSortCode.append("} else {");
        scriptSortCode.append("    return params.artists.length;");
        scriptSortCode.append("}");

        Script script = new Script(ScriptType.INLINE, "painless", scriptSortCode.toString(), params);

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder().query(boolQueryBuilder);
        searchSourceBuilder.sort(new ScriptSortBuilder(script, ScriptSortBuilder.ScriptSortType.NUMBER).order(SortOrder.ASC));
        searchSourceBuilder.sort("published_at", SortOrder.DESC);
        searchSourceBuilder.sort("id", SortOrder.DESC);
        searchSourceBuilder.size(size * 5); // 查询5倍的数据供后期歌手标签去重

        SearchRequest searchRequest = new SearchRequest().indices("data_music_news").types("music_data").source(searchSourceBuilder);

        try {
            Set<String> existedArtists = new HashSet<>();
            for (SearchHit searchHit : restHighLevelClient.search(searchRequest).getHits()) {
                ENews eNews = objectMapper.readValue(searchHit.getSourceAsString(), ENews.class);

                if (CollectionUtils.isNotEmpty(eNews.getArtists())) {
                    Set<String> retain = new HashSet<>(existedArtists);
                    retain.retainAll(eNews.getArtists());
                    if (CollectionUtils.isNotEmpty(retain)) {
                        continue;
                    }
                    existedArtists.addAll(eNews.getArtists());
                }

                eNewsList.add(eNews);
                if (eNewsList.size() >= size) {
                    break;
                }
            }
        } catch (IOException e) {
            LOGGER.error("Fetch ENews list from elasticsearch failed", e);
        }

        return eNewsList;
    }
```
在做第一版时，将热门歌手列表作为筛选条件，不在热门歌手列表内的结果全部舍弃。由于热门歌手的数量有一百多个，新闻的量有八万多，每次查询的时间复杂度大__（简直是Huge）__，于是在接口请求量大的时候导致CPU负载过高，查询速度慢，请求超时的情况。
![CPU Load](/img/es-cpu.png "CPU Load")

#### 改进版本
``` java
        scriptSortCode.append("if (doc['artists'].size() > 0 && new Date().getTime() - doc['published_at'].getValue().getMillis() < 86400000 * 3) {");
        scriptSortCode.append("    for (int i = 0; i < doc['artists'].length; ++i) {");
        scriptSortCode.append("        if (params.ranks.containsKey(doc['artists'][i])) {");
        scriptSortCode.append("             return params.ranks[doc['artists'][i]];");
        scriptSortCode.append("        }");
        scriptSortCode.append("    }");
        scriptSortCode.append("    return params.ranks.size();");
        scriptSortCode.append("} else {");
        scriptSortCode.append("    return params.ranks.size();");
        scriptSortCode.append("}");

```
由于原始版本的查询script使得CPU负载过高，所以将数量较少的新闻歌手关键词作为循环条件。

### Search API
参考:(Script Query)[https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-script-query.html]

在Kibana中使用Dev Tools进行查询时，如果按照上面java代码script中的"return params.ranks.size();"，查询会报错：
``` json
{
  "error": {
    "root_cause": [
      {
        "type": "script_exception",
        "reason": "runtime error",
        "script_stack": [
          "java.lang.Class.cast(Class.java:3369)",
          "return params.ranks[doc['artists'][i]];}}",
          "                   ^---- HERE"
        ],
        "script": "if (doc['artists'].size() > 0 && new Date().getTime() - doc['published_at'].getValue().getMillis() < 86400000 * 8){for (int i = 0; i < doc['artists'].length; ++i) {if (params.ranks.containsKey(doc['artists'][i])) { return params.ranks[doc['artists'][i]];}}return return params.ranks.size();;} else {return return params.ranks.size();;}",
        "lang": "painless"
      }
    ],
    "type": "search_phase_execution_exception",
    "reason": "all shards failed",
    "phase": "query",
    "grouped": true,
    "failed_shards": [
      {
        "shard": 0,
        "index": "data_music_news",
        "node": "3UC1bmxcQtO39l5XmCR93g",
        "reason": {
          "type": "script_exception",
          "reason": "runtime error",
          "script_stack": [
            "java.lang.Class.cast(Class.java:3369)",
            "return params.ranks[doc['artists'][i]];}}",
            "                   ^---- HERE"
          ],
          "script": "if (doc['artists'].size() > 0 && new Date().getTime() - doc['published_at'].getValue().getMillis() < 86400000 * 8){for (int i = 0; i < doc['artists'].length; ++i) {if (params.ranks.containsKey(doc['artists'][i])) { return params.ranks[doc['artists'][i]];}}return false;} else {return false;}",
          "lang": "painless",
          "caused_by": {
            "type": "class_cast_exception",
            "reason": "Cannot cast java.lang.Integer to java.lang.Boolean"
          }
        }
      }
    ]
  },
  "status": 500
}
```
大概意思就是返回值只接受布尔类型，于是将返回值修改后查询代码如下：
``` json
GET news/_search
{
  "query": {
        "bool": {
            "must": {
                "script" : {
                    "script" : {
                        "source" : "if (doc['artists'].size() > 0 && new Date().getTime() - doc['published_at'].getValue().getMillis() < 86400000 * 8){for (int i = 0; i < doc['artists'].length; ++i) {if (params.ranks.containsKey(doc['artists'][i])) { return true;}}return false;} else {return false;}",
                        "lang"   : "painless",
                        "params" : {
                            "ranks" : {
                              "Travis Scott": 1,
                              "Lady GaGa": 2,
                              "DJ Snake": 3,
                              "G-Eazy": 4
                            }
                        }
                    }
                }
            }
        }
    }
}
```