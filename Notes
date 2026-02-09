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