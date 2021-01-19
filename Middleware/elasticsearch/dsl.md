1.多条件聚合查询

```json
GET kibana_sample_data_ecommerce/_doc/_search
{
  "from":0,
  "size":0,
  "aggs": {
    "by_code": {
      "terms": {
        "script": "[doc.customer_gender.value, doc.day_of_week.value].join('-')",
        "size":14
      },
      "aggs": {
        "max-date": {
          "max": {
            "field": "order_date"
            }
        }
      }
    }
  }
}
```



2、计数

```json
GET kibana_sample_data_ecommerce/_doc/_search
{
  "size":0,
  "aggs": {
    "by_code": {
      "cardinality": {
        "script": "[doc.customer_gender.value, doc.day_of_week.value].join('-')"
      }
    }
  }
}
```

