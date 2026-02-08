# Elasticsearch ILM (Index Lifecycle Management)

## ILM Policy

### Complete ILM Policy with All Phases

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

## ILM Operations

### List All ILM Policies

```json
GET /_ilm/policy
```

### Get Specific ILM Policy

```json
GET /_ilm/policy/my-lifecycle-policy
```

### Delete ILM Policy

```json
DELETE /_ilm/policy/my-lifecycle-policy
```

### Explain ILM Status for Index

```json
GET /my-index/_ilm/explain
```

### Start ILM

```json
POST /_ilm/start
```

### Stop ILM

```json
POST /_ilm/stop
```

### Move Index to Next Step

```json
POST /my-index/_ilm/move
{
  "current_step": {
    "phase": "hot",
    "action": "complete",
    "name": "complete"
  },
  "next_step": {
    "phase": "warm",
    "action": "complete",
    "name": "complete"
  }
}
```

### Retry Failed ILM Step

```json
POST /my-index/_ilm/retry
```

### Remove ILM Policy from Index

```json
POST /my-index/_ilm/remove
```
