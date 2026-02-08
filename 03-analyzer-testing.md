# Testing Analyzers in Elasticsearch

## Basic Analyzer Testing

### Test Standard Analyzer

```json
POST /_analyze
{
  "analyzer": "standard",
  "text": "The Quick Brown Fox jumps over the lazy dog!"
}
```

### Test Custom Analyzer

```json
POST /_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Testing my custom analyzer with UPPERCASE and lowercase"
}
```

### Test Analyzer from Specific Index

```json
POST /my-index/_analyze
{
  "analyzer": "my_custom_analyzer",
  "text": "Sample text to analyze"
}
```

## Testing Individual Components

### Test Tokenizer Only

```json
POST /_analyze
{
  "tokenizer": "standard",
  "text": "Break this text into tokens"
}
```

### Test Tokenizer with Filters

```json
POST /_analyze
{
  "tokenizer": "standard",
  "filter": ["lowercase", "stop"],
  "text": "The QUICK brown fox"
}
```

### Test Character Filters

```json
POST /_analyze
{
  "char_filter": ["html_strip"],
  "tokenizer": "standard",
  "text": "<p>This is <b>HTML</b> text</p>"
}
```

## Scenario 1: Polish Language Analysis (ngram + lowercase)

### Create Index with Polish Analyzer

```json
PUT /products-pl
{
  "settings": {
    "analysis": {
      "analyzer": {
        "polish_ngram_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "polish_stop",
            "polish_ngram"
          ]
        },
        "polish_search_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "polish_stop"
          ]
        }
      },
      "filter": {
        "polish_ngram": {
          "type": "ngram",
          "min_gram": 3,
          "max_gram": 4
        },
        "polish_stop": {
          "type": "stop",
          "stopwords": ["i", "w", "z", "na", "do", "od"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "polish_ngram_analyzer",
        "search_analyzer": "polish_search_analyzer"
      },
      "description": {
        "type": "text",
        "analyzer": "polish_ngram_analyzer",
        "search_analyzer": "polish_search_analyzer"
      }
    }
  }
}
```

### Test Polish Ngram Analyzer (Index Time)

```json
POST /products-pl/_analyze
{
  "analyzer": "polish_ngram_analyzer",
  "text": "Laptop z ekranem dotykowym"
}
```

**Expected Result:**
```
["lap", "lapt", "apt", "apto", "pto", "ptop", "top", "ekr", "ekra", "kra", "kran", 
 "ran", "rane", "ane", "anem", "nem", "dot", "doty", "oty", "otyk", "tyk", "tyko", 
 "yko", "ykow", "kow", "kowy", "owy", "owym", "wym"]
```

### Test Polish Search Analyzer (Search Time)

```json
POST /products-pl/_analyze
{
  "analyzer": "polish_search_analyzer",
  "text": "laptop"
}
```

**Expected Result:**
```
["laptop"]
```

### Insert Sample Documents

```json
POST /products-pl/_bulk
{ "index": {} }
{ "name": "Laptop Dell z ekranem dotykowym", "description": "Profesjonalny laptop do pracy i rozrywki" }
{ "index": {} }
{ "name": "Tablet Samsung z rysikiem", "description": "Idealny tablet do rysowania i notowania" }
{ "index": {} }
{ "name": "Smartfon iPhone z aparatem", "description": "Telefon z najlepszym aparatem na rynku" }
```

### Test Partial Match Search

```json
GET /products-pl/_search
{
  "query": {
    "match": {
      "name": "lap"
    }
  }
}
```

**Expected:** Should find "Laptop Dell z ekranem dotykowym" (ngram matches "lap")

### Test Full Word Search

```json
GET /products-pl/_search
{
  "query": {
    "match": {
      "name": "laptop"
    }
  }
}
```

**Expected:** Should find "Laptop Dell z ekranem dotykowym"

### Test Stopword Filtering

```json
GET /products-pl/_search
{
  "query": {
    "match": {
      "name": "laptop z ekranem"
    }
  }
}
```

**Expected:** "z" (stopword) should be filtered out, search works on "laptop" and "ekranem"

## Scenario 2: Email and URL Tokenization (uax_url_email)

### Create Index with Email/URL Analyzer

```json
PUT /contacts
{
  "settings": {
    "analysis": {
      "analyzer": {
        "email_url_analyzer": {
          "type": "custom",
          "tokenizer": "uax_url_email",
          "filter": [
            "lowercase",
            "email_domain_filter"
          ]
        }
      },
      "filter": {
        "email_domain_filter": {
          "type": "pattern_capture",
          "preserve_original": true,
          "patterns": [
            "([^@]+)",
            "(\\p{L}+)",
            "@(.+)"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "email": {
        "type": "text",
        "analyzer": "email_url_analyzer",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "website": {
        "type": "text",
        "analyzer": "email_url_analyzer"
      },
      "notes": {
        "type": "text",
        "analyzer": "standard"
      }
    }
  }
}
```

### Test Email Tokenization

```json
POST /contacts/_analyze
{
  "analyzer": "email_url_analyzer",
  "text": "Contact me at john.doe@example.com or jane@company.co.uk"
}
```

**Expected Result:**
```
["john.doe@example.com", "john.doe", "john", "doe", "example.com",
 "jane@company.co.uk", "jane", "company.co.uk"]
```

### Test URL Tokenization

```json
POST /contacts/_analyze
{
  "analyzer": "email_url_analyzer",
  "text": "Visit https://www.example.com or http://blog.example.org/posts"
}
```

**Expected Result:**
```
["https://www.example.com", "http://blog.example.org/posts", ...]
```

### Insert Sample Documents

```json
POST /contacts/_bulk
{ "index": {} }
{ "email": "john.smith@gmail.com", "website": "https://johnsmith.dev", "notes": "Senior developer" }
{ "index": {} }
{ "email": "jane.doe@company.com", "website": "https://company.com/jane", "notes": "Product manager" }
{ "index": {} }
{ "email": "admin@example.org", "website": "https://example.org", "notes": "System administrator" }
```

### Search by Email Domain

```json
GET /contacts/_search
{
  "query": {
    "match": {
      "email": "gmail.com"
    }
  }
}
```

**Expected:** Should find john.smith@gmail.com

### Search by Email Username

```json
GET /contacts/_search
{
  "query": {
    "match": {
      "email": "jane"
    }
  }
}
```

**Expected:** Should find jane.doe@company.com

### Search by Full Email

```json
GET /contacts/_search
{
  "query": {
    "term": {
      "email.keyword": "john.smith@gmail.com"
    }
  }
}
```

**Expected:** Exact match for john.smith@gmail.com

### Search by Website Domain

```json
GET /contacts/_search
{
  "query": {
    "match": {
      "website": "example.org"
    }
  }
}
```

**Expected:** Should find admin@example.org

### Complex Query: Email OR Website

```json
GET /contacts/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "email": "company" } },
        { "match": { "website": "company" } }
      ]
    }
  }
}
```

**Expected:** Should find jane.doe@company.com (matches both email and website)

## Comparison Testing

### Compare Multiple Analyzers

```json
POST /_analyze
{
  "analyzer": "standard",
  "text": "user@example.com"
}

POST /_analyze
{
  "tokenizer": "uax_url_email",
  "text": "user@example.com"
}
```

### Test Edge Cases

```json
POST /contacts/_analyze
{
  "analyzer": "email_url_analyzer",
  "text": "Multiple emails: test@test.com, admin@admin.org, and contact@site.co.uk"
}
```

## Quick Testing Snippets

### Test Before Creating Index

```json
# Test your analyzer configuration before creating index
POST /_analyze
{
  "tokenizer": "standard",
  "filter": ["lowercase", "stop"],
  "text": "Your sample text HERE"
}
```

### Test Existing Index Analyzer

```json
# Quick test of existing index analyzer
POST /your-index/_analyze
{
  "field": "your_field",
  "text": "Test this text"
}
```

### Compare Index vs Search Analyzer

```json
# Test index analyzer
POST /products-pl/_analyze
{
  "analyzer": "polish_ngram_analyzer",
  "text": "laptop"
}

# Test search analyzer
POST /products-pl/_analyze
{
  "analyzer": "polish_search_analyzer",
  "text": "laptop"
}
```
