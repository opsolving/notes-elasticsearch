# notes-elasticsearch

## Tworzenie indeksów

### Podstawowy indeks

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

### Indeks z analizatorem

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

### Sprawdzenie istnienia indeksu

```json
HEAD /my-index
```

### Lista wszystkich indeksów

```json
GET /_cat/indices?v
```

### Usunięcie indeksu

```json
DELETE /my-index
```

### Pobierz mapowanie indeksu

```json
GET /my-index/_mapping
```

### Pobierz ustawienia indeksu

```json
GET /my-index/_settings
```

### Aktualizacja ustawień indeksu

```json
PUT /my-index/_settings
{
  "index": {
    "number_of_replicas": 2
  }
}
```

### Dodaj nowe pole do istniejącego mapowania

```json
PUT /my-index/_mapping
{
  "properties": {
    "new_field": { "type": "keyword" }
  }
}
```

## Index Templates

### Prosty index template

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

### Lista wszystkich templates

```json
GET /_index_template
```

### Usuń template

```json
DELETE /_index_template/my-template
```

## Component Templates

### Component template dla mappings

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

### Component template dla settings

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

### Index template używający component templates

```json
PUT /_index_template/combined-template
{
  "index_patterns": ["app-*"],
  "composed_of": ["my-settings", "my-mappings"],
  "priority": 500
}
```

## ILM (Index Lifecycle Management)

### ILM Policy

```json
PUT /_ilm/policy/my-lifecycle-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "30d"
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

### Lista ILM policies

```json
GET /_ilm/policy
```

### Component template z ILM

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

### Index template dla data streams

```json
PUT _index_template/logs-template
{
  "index_patterns": ["logs-*"],
  "data_stream": {},
  "composed_of": ["logs-mappings"],
  "priority": 100
}
```

### Tworzenie data stream

```json
PUT _data_stream/logs-app
```

## Aliases

### Utwórz alias

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

### Alias z filtrem

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

### Lista aliasów

```json
GET /_cat/aliases?v
```

### Usuń alias

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

### Podstawowy reindex

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

### Reindex z query

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

### Reindex z transformacją

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

### Statystyki indeksu

```json
GET /my-index/_stats
```

### Rozmiar indeksu

```json
GET /_cat/indices/my-index?v&h=index,store.size,pri.store.size
```

### Liczba dokumentów

```json
GET /my-index/_count
```
