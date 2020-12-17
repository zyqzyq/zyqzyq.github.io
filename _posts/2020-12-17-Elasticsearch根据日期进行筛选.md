---
author: zyqzyq
date: 2020-12-17 10:10:32+00:00
layout: post
title: Elasticsearch根据日期进行筛选
key: 20201217
tags:
- elasticsearch
---



# Elasticsearch根据日期进行筛选



#### 创建索引

```
PUT index_name
```

#### 设置mapping

```
PUT /index_name/_mapping
{
      "properties": {
        "create_time": {
          "type":   "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
        }
  }
}
```

#### 进行时间范围筛选

多条件筛选

```
GET /index_name/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "create_user": "xxx"
          }
        },
        {
          "range": {
            "create_time": {
              "gte": "2020-12-14 10:45:11",
              "lte": "now",
              "format": "yyyy-MM-dd HH:mm:ss"
            }
          }
        }
      ]
    }
  }
}
```

