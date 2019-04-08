---
title: Elasticsearch 模糊查询(Fuzzy Query) 与编辑距离算法(Edit Distance)
cdn: header-off
date: 2018-12-27 16:50:17
header-img: "/img/es.png"
tags:
    - Elasticsearch
---

## Fuzzy Matching
  在ES的[Match Query](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/query-dsl-match-query.html)中，有一个[fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/common-options.html#fuzziness)字段可以对搜索关键词进行模糊匹配。fuzziness的值可以指定为固定的数值如(0, 1, 2)或是数值区间如0...2, 3...5, >5等，数值的含义原文为：
  > When querying text or keyword fields, fuzziness is interpreted as a Levenshtein Edit Distance — the number of one character changes that need to be made to one string to make it the same as another string.
  
  大致意思是，搜索关键词可根据指定数值的编辑距离（Levenshtein Edit Distance）进行变换进行关键词的模糊查询，每一次距离的变换可以理解为字符串中一个字符的一次插入、删除或替换操作。

### Java
``` java
private Page<Artist> searchArtist0(String word, Pageable pageable) {
        if (StringUtils.isBlank(word)) {
            return new PageImpl<>(Collections.emptyList(), pageable, 0);
        }

        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        boolQueryBuilder.must(QueryBuilders.matchQuery("name", word).fuzziness(1));
        if (CollectionUtils.isNotEmpty(musicBlacklistService.artists())) {
            boolQueryBuilder.mustNot(QueryBuilders.termsQuery("id", musicBlacklistService.artists()));
        }

        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        searchSourceBuilder.query(boolQueryBuilder);
        searchSourceBuilder.size(pageable.getPageSize());
        searchSourceBuilder.from(pageable.getPageSize() * pageable.getPageNumber());

        searchSourceBuilder.fetchSource(new String[]{"id", "name", "thumbnails"}, new String[]{});

        SearchRequest searchRequest = new SearchRequest().indices("data_artist_v2").types("music_data").source(searchSourceBuilder);

        try {
            SearchHits searchHits = restHighLevelClient.search(searchRequest).getHits();

            List<Artist> artists = new ArrayList<>();
            for (SearchHit searchHit : searchHits) {
                artists.add(objectMapper.readValue(searchHit.getSourceAsString(), Artist.class));
            }

            return new PageImpl<>(artists, pageable, searchHits.getTotalHits());
        } catch (IOException e) {
            LOGGER.error("Search artists with word:{} from elasticsearch error:{}", word, e);
        }

        return new PageImpl<>(Collections.emptyList(), pageable, 0);
    }
```

### ES JSON
``` json
GET /data_artist_v2/music_data/_search
{
  "from": 0,
  "size": 5,
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": {
              "query": "max",
              "fuzziness": 1
            }
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "id": {
              "value": "VALUE"
            }
          }
        }
      ]
    }
  },
  "_source": [
    "id",
    "name",
    "thumbnails"
  ]
}
```

## Levenshtein Edit Distance
### Definition
> the Levenshtein distance is a string metric for measuring the difference between two sequences. Informally, the Levenshtein distance between two words is the minimum number of single-character edits (insertions, deletions or substitutions) required to change one word into the other. 

### 计算法则
以下是字符串a、b之间相互转换所需要的最小编辑距离的计算公式：
![Levenshtein Distance Fomula](/img/levenshtein-formula.png "Levenshtein Distance Fomula") 

根据维基百科里的[Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance)公式，可以用一个二维数组标识string a到string b的最小编辑距离，例如：

|       |   | k | i | t | t | e | n |  
|:-----:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|       | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| **s** | 1 | 1 | 2 | 3 | 4 | 5 | 6 |
| **i** | 2 | 2 | 1 | 2 | 3 | 4 | 5 | 
| **t** | 3 | 3 | 2 | 1 | 2 | 3 | 4 | 
| **t** | 4 | 4 | 3 | 2 | 1 | 2 | 3 | 
| **i** | 5 | 5 | 4 | 3 | 2 | 2 | 3 | 
| **n** | 6 | 6 | 5 | 4 | 3 | 3 | 2 | 
| **g** | 7 | 7 | 6 | 5 | 4 | 4 | 3 | 

所以**kitten**到**sitting**的最小编辑距离为3。


### § LeetCode - 72. Edit Distance
LeetCode中有有一道运用Levenshtein Distance来解的题，[72. Edit Distance](https://leetcode.com/problems/edit-distance/)
#### § Detail
Given two words word1 and word2, find the minimum number of operations required to convert word1 to word2.
You have the following 3 operations permitted on a word:
1. Insert a character
2. Delete a character
3. Replace a character

#### § Example
1. 
> **Input**: word1 = "horse", word2 = "ros"
> **Output**: 3
> **Explanation**: 
> horse -> rorse (replace 'h' with 'r')
> rorse -> rose (remove 'r')
> rose -> ros (remove 'e')

2. 
> **Input**: word1 = "intention", word2 = "execution"
> **Output**: 5
> **Explanation**: 
> intention -> inention (remove 't')
> inention -> enention (replace 'i' with 'e')
> enention -> exention (replace 'n' with 'x')
> exention -> exection (replace 'n' with 'c')
> exection -> execution (insert 'u')

#### § Solution
``` java
    public static int minDistance(String word1, String word2) {
        if(word1.equals("") && word2.equals("")) {
            return 0;
        } else if(word1.equals("") || word2.equals("")) {
            if(word1.equals("")) {
                return word2.length();
            } else {
                return word1.length();
            }
        } else {
            int[][] leveintash = new int[word1.length() + 1][word2.length() + 1];
            for (int i = 0; i <= word2.length(); i++) {
                leveintash[0][i] = i;
            }
            for (int i = 0; i <= word1.length(); i++) {
                leveintash[i][0] = i;
            }
            for (int i = 1; i <= word2.length(); i++) {
                for (int j = 1; j <= word1.length(); j++) {
                    int minimum = Math.min(Math.min(leveintash[j - 1][i - 1], leveintash[j - 1][i]), leveintash[j][i - 1]) + 1;
                    if (word1.charAt(j - 1) == word2.charAt(i - 1))
                        minimum = leveintash[j - 1][i - 1];
                    leveintash[j][i] = minimum;
                }
            }
            print2DArray(word1, word2, leveintash);
            return leveintash[word1.length()][word2.length()];
        }
    }
```