---
title: 使用Sentinl通过access_log监控接口耗时占比同比增长率
cdn: header-off
date: 2018-10-12 15:32:07
header-img: "/img/sentinl.jpg"
tags:
    - Elasticsearch
---
> SENTINL extends Kibi and Kibana with Alerting and Reporting functionality to monitor, notify and report on data series changes using standard queries, programmable validators and a variety of configurable actions - Think of it as a free an independent "Watcher" and "Reporting" alternative, further extended and expanded by the unique Kibi features
>
>SENTINL is also designed to simplify the process of creating and managing alerts and reports in Kibi/Kibana via its integrated App and Spy integration.

## 需求背景
在对接口耗时进行接口告警的时候，有些接口希望通过前一天同一时段的相对较长的响应时间占比来进行对比，超过阈值的则进行告警。

## 实现方式
Elasticsearch提供了[百分比排行聚合(Percentile Ranks Aggregation)](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/search-aggregations-metrics-percentile-rank-aggregation.html)的聚合方式，因此在做同比占比统计的时候，可以先规定一个标准响应时间值，统计该标准响应时间的排行百分比，并且比较该排行百分比。

### 原始数据
解析后的access_log数据大致如下：
``` json
{
  "took": 1295,
  "timed_out": false,
  "_shards": {
    "total": 35,
    "successful": 35,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 482,
    "max_score": 6.050335,
    "hits": [
      {
        "_index": "go_music_access-2018.10.12",
        "_type": "doc",
        "_id": "yeJ_Z2YBG7DfNsy-6v8J",
        "_score": 6.050335,
        "_source": {
          "params": "api_key=&device=&access_token=",
          "request_time": "12/Oct/2018:16:58:30 +0800",
          "@timestamp": "2018-10-12T08:58:47.930Z",
          "server_ip": "10.10.33.52",
          "uri": "/api/v1/playlists",
          "host": "pmuswebs03.la.gomo.com",
          "http_status": "200",
          "response_time": "13",
          "geoip": {
            "latitude": 37.1784,
            "ip": "83.45.35.176",
            "postal_code": "18001",
            "region_code": "GR",
            "city_name": "Granada",
            "continent_code": "EU",
            "longitude": -3.5991999999999997,
            "timezone": "Europe/Madrid",
            "country_code2": "ES",
            "country_name": "Spain",
            "region_name": "Granada",
            "location": {
              "lat": 37.1784,
              "lon": -3.5991999999999997
            },
            "country_code3": "ES"
          },
          "size": "148",
          "method": "POST",
          "client_ip": "83.45.35.176"
        }
      },
      ...
    ]
  }
}
```

### 匹配方式
规定500ms为标准响应时间，若当前时段500ms以上响应时间占比高于昨日同一时段500ms以上响应时间占比的50%，则告警。
``` json
{
  "actions": {
    "Webhook_b3a830b2-7993-4a5d-8e6e-499552b0e6be": {
      "name": "",
      "throttle_period": "1m",
      "webhook": {
        "priority": "high",
        "stateless": false,
        "method": "POST",
        "host": "",
        "port": "80",
        "path": "/v2/alarms/zJjcoDxszISxHtZUVOcMGcJwDQjkYifl",
        "body": "{\"contact\": \"\",\"values\": {\"title\": \"\",\"content\": \"\",\"url\":\"\"}}",
        "headers": {
          "Content-Type": "application/json;charset=UTF-8"
        },
        "auth": "",
        "message": ""
      }
    }
  }
  "input": {
    "search": {
      "request": {
        "index": [
          "go_music_access-*"
        ],
        "body": {
          "query": {
            "bool": {
              "filter": [
                {
                  "bool": {
                    "should": [
                      {
                        "range": {
                          "@timestamp": {
                            "gte": "now-1d-1h",
                            "lte": "now-1d",
                            "format": "epoch_millis"
                          }
                        }
                      },
                      {
                        "range": {
                          "@timestamp": {
                            "gte": "now-5m",
                            "lte": "now",
                            "format": "epoch_millis"
                          }
                        }
                      }
                    ]
                  }
                }
              ],
              "must": [
                {
                  "regexp": {
                    "uri": {
                      "value": "/api/v3/albums/[0-9]+"
                    }
                  }
                },
                {
                  "term": {
                    "http_status": {
                      "value": 200
                    }
                  }
                }
              ]
            }
          },
          "size": 0,
          "aggs": {
            "last": {
              "filter": {
                "range": {
                  "@timestamp": {
                    "gte": "now-1d-1h",
                    "lte": "now-1d"
                  }
                }
              },
              "aggs": {
                "percentageAgg": {
                  "percentile_ranks": {
                    "field": "response_time",
                    "values": [
                      500
                    ]
                  }
                }
              }
            },
            "current": {
              "filter": {
                "range": {
                  "@timestamp": {
                    "gte": "now-5m",
                    "lte": "now"
                  }
                }
              },
              "aggs": {
                "percentageAgg": {
                  "percentile_ranks": {
                    "field": "response_time",
                    "values": [
                      500
                    ]
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "condition": {
    "script": {
      "script": "payload.aggregations.last.percentageAgg.values['500.0'] - payload.aggregations.current.percentageAgg.values['500.0'] > 50"
    }
  },
  "trigger": {
    "schedule": {
      "later": "every 5 minutes"
    }
  },
  "disable": false,
  "report": true,
  "title": "",
  "wizard": {},
  "save_payload": true,
  "spy": true,
  "impersonate": false
}
```
