----
page_title: Fixed Schema Elasticsearch
page_description: Fixed Schema Elasticsearch grafana datasource documentation
page_keywords: Elasticsearch, grafana, kibana, documentation, datasource, docs
---

# Fixed Schema Elasticsearch

This datasource extends the [Elasticsearch](/datasources/elasticsearch/)
datasource, imposing some constrains on the way you store you data. This allows
more precise auto-completion and more performant queries.

In particular, it modifies the auto-completion such that:
- one can autocomplete on the metric names only (and not all the fields in
elastic)
- when a metric is selected, the tags autocompletes only for the tags
associated with this metric

# How should you store your data ?

- Every metric type (e.g. cpu, memory) should be in a single type within the
elasticsearch Index. 
- Your model for your data in elastic must be flat (i.e. no nested fields) as
in the Elasticsearch datasource.
- Every document in elastcisearch can store only one metric e.g. it is not
yet possible to display
```
{
    "timestamp": 1442165810,
    "cpu": 1.2, 
    "memory": 368873
    "host": "my_hostname",
    "instance": "Local",
}
```

Only the first constrain is hard, and some developement could lift the two
others.

/!\ The auto-completion is based on the mapping of the index of the current
day. It supposes that the tags keys (not the values) of your metrics has not
changed over time.

## Configure the Elasticsearch index

For intance, if you want to stores your data on indexes starting with 
`test-metrics`, you ca use the following template for elasticsearch.
The data field in elasticsearch is the one you will use when adding the
datasource

```
PUT _template/metrics_template
{
  "template": "test-metrics*",
  "settings": {
    "index": {
      "refresh_interval": "5s"
    }
  },
  "mappings": {
    "_default_": {
      "dynamic_templates": [
        {
          "strings": {
            "match": "*",
            "match_mapping_type": "string",
            "mapping":   { "type": "string",  
                           "doc_values": true, 
                           "index": "not_analyzed" }
          }
        }
      ],
      "_all":            { "enabled": false },
      "properties": {
        "timestamp":    { "type": "date",    "doc_values": true}
      }
    }
  }
}
```

One can see [this blog](https://www.elastic.co/blog/elasticsearch-as-a-time-series-data-store) for more details on the template configuration.

## Adding the data source
![](/img/v2/add_Graphite.jpg)

Simply follow the same instructions as in 
[Elasticsearch](/datasources/elasticsearch/)

