# Elasticsearch Index Management

## Creating Indexes

### Basic Index

```json
PUT /my-index
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "created_at": { "type": "date" },
      "status": { "type": "keyword" }
    }
  }
}
```

### Index with Custom Analyzer

```json
PUT /my-index-with-analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "stop", "snowball"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}
```

### Check Index Existence

```json
HEAD /my-index
```

### List All Indexes

```json
GET /_cat/indices?v
```

### Delete Index

```json
DELETE /my-index
```

### Get Index Mapping

```json
GET /my-index/_mapping
```

### Get Index Settings

```json
GET /my-index/_settings
```

### Update Index Settings

```json
PUT /my-index/_settings
{
  "index": {
    "number_of_replicas": 2
  }
}
```

### Add New Field to Existing Mapping

```json
PUT /my-index/_mapping
{
  "properties": {
    "new_field": { "type": "keyword" }
  }
}
```

## Index Templates

### Simple Index Template

```json
PUT /_index_template/my-template
{
  "index_patterns": ["my-index-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "user": { "type": "keyword" },
        "message": { "type": "text" }
      }
    }
  },
  "priority": 200
}
```

### List All Templates

```json
GET /_index_template
```

### Delete Template

```json
DELETE /_index_template/my-template
```

## Component Templates

### Component Template for Mappings

```json
PUT /_component_template/my-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "ip_address": { "type": "ip" },
        "user_agent": { "type": "text" }
      }
    }
  }
}
```

### Component Template for Settings

```json
PUT /_component_template/my-settings
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "refresh_interval": "30s"
    }
  }
}
```

### Index Template Using Component Templates

```json
PUT /_index_template/combined-template
{
  "index_patterns": ["app-*"],
  "composed_of": ["my-settings", "my-mappings"],
  "priority": 500
}
```

## Data Streams

### Component Template with ILM

```json
PUT _component_template/logs-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "level": { "type": "keyword" },
        "message": { "type": "text" }
      }
    },
    "settings": {
      "index.lifecycle.name": "my-lifecycle-policy"
    }
  }
}
```

### Index Template for Data Streams

```json
PUT _index_template/logs-template
{
  "index_patterns": ["logs-*"],
  "data_stream": {},
  "composed_of": ["logs-mappings"],
  "priority": 100
}
```

### Create Data Stream

```json
PUT _data_stream/logs-app
```

## Aliases

### Create Alias

```json
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "my-index-000001",
        "alias": "my-alias"
      }
    }
  ]
}
```

### Alias with Filter

```json
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "logs-*",
        "alias": "error-logs",
        "filter": {
          "term": { "level": "error" }
        }
      }
    }
  ]
}
```

### List Aliases

```json
GET /_cat/aliases?v
```

### Delete Alias

```json
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "my-index-000001",
        "alias": "my-alias"
      }
    }
  ]
}
```

## Reindex

### Basic Reindex

```json
POST /_reindex
{
  "source": {
    "index": "old-index"
  },
  "dest": {
    "index": "new-index"
  }
}
```

### Reindex with Query

```json
POST /_reindex
{
  "source": {
    "index": "old-index",
    "query": {
      "range": {
        "@timestamp": {
          "gte": "2024-01-01"
        }
      }
    }
  },
  "dest": {
    "index": "new-index"
  }
}
```

### Reindex with Transformation

```json
POST /_reindex
{
  "source": {
    "index": "old-index"
  },
  "dest": {
    "index": "new-index"
  },
  "script": {
    "source": "ctx._source.new_field = ctx._source.old_field * 2"
  }
}
```

## Index Stats

### Index Statistics

```json
GET /my-index/_stats
```

### Index Size

```json
GET /_cat/indices/my-index?v&h=index,store.size,pri.store.size
```

### Document Count

```json
GET /my-index/_count
```
