# API Query Reference

`GET /` Get's basic system/cluster data

`GET _cat/indices/` Get's a list of indexes (including system indexes)

`GET _cat/indices/*,-.*?v` Get list of non system indexes

`GET kibana_sample_data_ecommerce/_search` Get list of 10 documents from index

`GET /matt-index/_search?q=rocky mountain` Keyword search in matt-index

# Indices

> Note: Indexes must be named with lowercase characters

`PUT /blogs` Create blogs index

`GET /_cat/indicies/blogs?v&h=health,status,index,docs.count,store.size&s=index:desc` Verify blogs index was created by listing these attributes of the index

`GET /_cat/indices?v&h=health,status,index,docs.count,store.size&s=index:desc` Get a list of all indexes

`DELETE blogs` Permanently deletes an index

# Document Manipulation
PUT - Operations will insert docs with a user specified document ID. The operation is idompotent, but requires that doc ID and documents are fully replaced. No partial update option is available.

POST - Operation will insert docs with a system generated doc ID. Operation is not idompotent. If a doc ID is specified and already exists the operation will fail. Partial updates are available with POST operations.

**Create doc with _id number of 1 - idompotent**
```
PUT /blogs/_doc/1
{
  "name": "Cola",
  "species": "dog"
}
```

**Create doc with an elasticsearch generate ID number**
```
POST /blogs/_doc
{
  "name": "Echo",
  "species": "dog"
}
```
**Create doc with a specified _id field - this will fail if the doc already exists**
```
POST /blogs/_create/iaVYB5wBMieTKl4qjWwf
{
  "name": "Echo",
  "species": "dog"
}
```


**Perform an update (partial update)**
```
POST /blogs/_update/iaVYB5wBMieTKl4qjWwf
{
  "doc":
    {
      "breed": "Shepherd"
    }
}
```

# Document Search
Search occurs on data-streams and indices.

`GET /blogs/_search` Queries and returns the first 10 docs from the blogs index

Below is the equivalent of the command above

```
GET /blogs/_search
{

  "query": {
    "match_all": {}
  }
}
```
# Aggregations

This is a basic aggregation to search the blogs index and search for the minimum value int he `publish_date` field. Giving you the frist blob that was posted.

```
GET blogs/_search
{
  "aggs": {
    "first_blog": {
      "min": {
        "field": "publish_date"
      }
    }
  }
}
```

# ES|QL Elastic Search Query Language

This is a bas ES|QL query that takes all results from the blogs index and keeps the publish_date, url, and names fields. Sorts them by publish_date and returns 10 of the queries.

```
FROM blogs*
| KEEP publish_date, url, authors.first_name, authors.last_name
| SORT publish_date
| LIMIT 10
```

Basic ES|QL API query, you can change the output format using the syntax below.

```
POST /_query?format=csv
{
  "query": "FROM blogs | KEEP publish_date, authors.last_name | SORT (publish_date)"
}
```

# Data Streams

| Term| Definition |
|-|-|
| Rollover | The operation that creates a new backing index for a data stream, applying the current template configuration. |
| constant_keyword| A field type optimized for fields where all documents in an index share the same value, reducing storage and improving query performance.|
| Mapping Immutability | The principle that once a field's mapping is defined in an index, it cannot be changed-only new indices can receive updated mappings. |

> Data Streams are groups of indices which are managed by index templates and component templates.

`POST metrics-custom-labenv/_rollover` Manually roll over data stream after committing changes to Component template mappings. This is required (maybe for index templates too) for data stream mappings to be updated

`GET metrics-custom-labenv/_mapping` View mappings of backend data stream

`GET metrics-custom-lavenv/_search` Search data stream

Below is a search query for a data-stream that searches for a particuar value

```
{
  "query": {
    "match": {
      "data_stream.type" : "metrics"
    }
  }
}
```

# Elastic Agent Integration

| Term | Definition |
|-|-|
| Integration | A pre-built package that configures Elastic Agent to collect specific data types with appropriate templates, pipelines, and dashboards |
| Namespace | The third component of the data stream name ({type}-{dataset}-{namespace}) used to separate data by environment, teamn, or deployment|
| Agent Policy| A configuration that defines which integrations and settings apply to a group of Elastic Agents |

> Names become part of the data stream name. It's a useful way to segment data sources so data does not get mixed. For example, this lab is using "labenv" as a namespace for the system integration. When building a home lab you may use something similar when testing a new data source.



# Data Management Concepts

Ask ChatGPT what rollover is and why it's important. It will explain how rollover is the (typically automated) function that creates new indices when they get too large.An index is given an alias such as `my-metrics` and from the standpoint of the data forwarder, that never changes. but Elastic on the background is indexing the data in my-metrics-000001. After that index gets 7 days old, or 50GB in size, or whatever the rollover/ILM policy dictates, Elastic then creates my-metrics-000002 which all new data being sent to my-metrics is indexed to. This keeps indicies from growing too large and slowing down queries.

## Key Terms

| Term | Definition |
|-|-|
| Component template	| A reusable building block containing mappings, settings, or aliases that can be combined into index templates |
| Index template	| A configuration that automatically applies settings, mappings, and aliases to new indices matching a specified pattern |
| Index pattern	| A wildcard expression (e.g., my-metrics-*) that determines which indices an index template applies to |
| Priority	| A numeric value that determines which template takes precedence when multiple templates match the same index name |
| Simulate Index API	| An API that shows what settings, mappings, and aliases would be applied to an index without actually creating it |


API Command to Create a Component Template

```json
PUT _component_template/time-series-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "status": {
          "type": "keyword"
        },
        "message": {
          "type": "text"
        }
      }
    }
  }
}
```

API Command to create index using a component template

```
PUT _index_template/my-metrics-template
{
  "priority": 500,
  "index_patterns": [
    "my-metrics-*"
  ],
  "composed_of": [
    "time-series-mappings"
  ]
}
```

`POST _index_template/_simulate_index/my-metrics-test` This command will give you what an index template would look like if it were to be created using the name `my-metrics-test` but doesn't actually created it.

Here is how to create another component template while specifying the number of replicas. Replicas are duplicates of the original copy of data. Useful for integrity of data in the event nodes in a cluster go down. Each replica is a full copy of the data.

PUT _component_template/time-series-settings
{
  "template": {
    "settings": {
      "index": {
        "number_of_replicas": "2"
      }
    }
  }
}

# Index Alises and Write Indexes

In this lesson, you'll create an index with an alias and configure it as the write index. This matters because aliases allow applications to write to a single, stable endpoint while Elasticsearch transparently routes documents to the correct backing index—enabling seamless index rollovers without application changes.

| Term	| Definition |
|-|-|
| Index alias	| A secondary name that points to one or more indices, allowing applications to reference indices without knowing their actual names |
| Index rollover	| Creates a new write index when the current one reaches a certain size, number of docs, or age and can target a data stream or an alias with a write index |
| Write index	| The single index within an alias that receives all indexing (write) operations; required when an alias points to multiple indices |
| Backing index	| An actual index that an alias points to; an alias can have multiple backing indices for reads but only one write index |

The naming convention -000001 is an Elasticsearch best practice for rollover indices. When you call the rollover API, Elasticsearch automatically increments the numeric suffix (000001 → 000002). Using this pattern from the start enables automated index lifecycle management.

This creates an index that is configured to be the write index for the my-metrics alias
```
PUT my-metrics-000001
{
    "aliases": {
        "my-metrics": {
            "is_write_index": true
        }
    }
}
```

Verify new index has been created and has appropriate settings.

GET my-metrics-000001
POST _index_template/_simulate_index/my-metrics-000001

# Index Rollover

In this challenge, you'll perform a rollover operation to create a new index and redirect writes to it. This matters because rollover is the foundation of time-series data management in Elasticsearch—it allows you to create new indices based on size, age, or document count conditions while maintaining a stable alias for applications.

| Term	| Definition |
|-|-|
| Rollover	| An operation that creates a new index and transfers the write index designation from the old index to the new one |
| Rollover conditions	Criteria | (max_age, max_docs, max_size, max_primary_shard_size) that determine when a rollover should occur |
| Index Lifecycle Management (ILM)	| A feature that automates rollover and other index operations based on policies; manual rollover is typically used for testing or one-off operations |

```
POST my-metrics/_rollover
{
  "conditions": {
    "max_age": "2s"
  }
}
```