---
layout: post
title: "Elasticsearch: function score, multi_match, fuzzy_match"
description: ""
categories: [elasticsearch]
tags: []
redirect_from:
  - /2019/06/19/
---

[How to use function score](https://qiita.com/yosu/items/b2d837ebe95049e9556e)

Problem to solve:

The original query
~~~json
GET maps/map/_search
{ "_source": "name_ja",
  "sort":"_score",
  "size":10,
  "query":{
    "bool":{
      "should":[
        { "match": {
          "name_kana.kana_romaji": {
            "query": "akadake",
            "fuzziness": "AUTO",
            "operator": "and"
          }
        }
     },{
         "match":{
           "name_ja.autocomplete":{
             "query":"akadake"
           }
          }
        },{
        "match":{
          "name_ja.readingform":{
            "query":"akadake",
            "fuzziness":"AUTO",
            "operator": "and"
           }
         }
       }]
      }
    }
  }
~~~
produced too low score on `name_kana.kana_romaji` field.

Using function_score：

[query reference](https://stackoverflow.com/questions/22830492/mixing-bool-and-multi-match-function-score-query)

~~~json
GET maps/map/_search
{ "_source": "name_ja",
  "sort":"_score",
  "size":10,
  "query":{
    "function_score": {
      "query": {
        "bool":{
          "should":[
            { "match": {
              "name_kana.kana_romaji": {
                "query": "akadake",
                "fuzziness": "AUTO",
                "operator": "and"
              }
            }
           }, {
           "match":{
             "name_ja.autocomplete":{
               "query":"akadake"
             }
            }
          },{
          "match":{
            "name_ja.readingform":{
              "query":"akadake",
              "fuzziness":"AUTO",
              "operator": "and"
             }
           }
         }]
        }
      },
      "boost": "6",
      "boost_mode": "multiply"
    }
  }
}
~~~
All fields multiply to 6, not solve the problem.
Turning to `multi_match` for specifying different weight to `name_kana.kana_romaji` only.

[`multi_match` and `match` both can do a `fuzzy` match](https://www.elastic.co/guide/en/elasticsearch/guide/current/fuzzy-match-query.html)

[query reference](https://stackoverflow.com/questions/49719337/function-score-with-multi-match-query)

Final result：
~~~json
GET maps/map/_search
{ "_source": "name_ja",
  "sort":"_score",
  "size":10,
  "query":{
    "bool":{
      "must":[
        { "multi_match": {
            "query": "akadake",
            "fields": [
              "name_ja.readingform",
              "name_ja.autocomplete",
              "name_kana.kana_romaji^4"
            ],
         "fuzziness": 1
         }
        }]
    }
  }
}
~~~
