# ElasticSearch Notes

## `Section5` Searching

### `5.1` Term Level Queries

    GET /products/_search
    {
        "query": {
            "term": {
                "brand.keyword": "Nike"
            }
        }
    }

* Term level queries are `not analyzed`, used for `exact matching`, entire term must match, not partly. 
* By default, `case sensitive`.    
* For `inverted index` lookups  
* Can be used with `keywords`, `numbers`, `dates`, `booleans`, ...  
* Don't use with `text`!

    GET /products/_search
    {
        "query": {
            "terms": {
                "tags.keyword": ["Soup", "Meat"]
            }
        }
    }

* Use `terms` for multiple terms  


### `5.2` Retrieve by IDs

    GET /products/_search
    {
        "query": {
            "ids": {
                "values": ["100", "200", "300"]
            }
        }
    }

### `5.3` Query by ranges
    
    GET /products/_search
    {
        "query": {
            "range": {
                "created": {
                    "time_zone": "+01:00"
                    "gte": "2020/01/01 01:00:00",
                    "lte": "2020/01/31 00:59:59"
                }
            }
        }
    }

- Datetime with format YYMMDD T hhmmss Z is from UTC zone.
- `time_zone` can compare different time zones

### `5.4` Prefix, Wildcard, Regular Expression

- Prefix  
    - Only check the beginning  
-

    GET /products/_search
    {
        "query": {
            "prefix": {
                "name.keyword": {
                    "value": "Past" 
                }
            }
        }
    }

- Wildcard  
    - `?` for single char, `*` for none or multiple chars  
- 

    GET /products/_search
    {
        "query": {
            "wildcard": {
                "tags.keyword": {
                    "value": "*Bee?" 
                }
            }
        }
    }

- Regular Expressions (regexp, for full string match)
    - `Bee(f|r)+`, starts with "Bee", followed by at least one "f" or "r", `+` means one or more

    - `Bee[a-zA-Z]+`, any letters allowed followed "Bee"

    - `Bee(r|t){1}`, only 1 "r" or "t" followed

    - `Beer.*`, `.` means any char, `*` means any times
- 

    GET /products/_search
    {
        "query": {
            "regexp": {
                "tags.keyword": {
                    "value": "Bee.*" 
                }
            }
        }
    }

- The above syntax is for `Apache Lucene`, different than other engines' regexp like `^, $, ...`

- All of `prefix`, `wildcard`, `regexp` can use `"case_insensitive": true`.

### `5.5` Querying by Field Existence

https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-exists-query.html

    GET /products/_search
    {
        "query": {
            "exists": {
                "field": "tags.keyword"
            }
        }
    }

- No indexed value
    - 1 `Null, []` (empty array) are not indexed as empty value
    - 2 But `""` (empty string) is not an empty value
    - 3 `index` mapping parameter is set to false
    - 4 value's length exceeds the `ignore_above` parameter
    - 5 malformed values (string -> numeric) with `ignore_malformed` parameter set to `true`

- the following for `not existing`:
-

    GET /products/_search
    {
        "query": {
            "bool": {
                "must_not": [
                    {
                        "exists": {
                            "field": "tags.keyword"
                        }
                    }
                ]
            }
        }
    }
    
    // similar to SQL the following:
    SELECT * FROM products WHERE tags IS NULL

### `5.6` Full text queries

- `Term level queries` for `exact matching` on structured data, and `not analyzed`
- diffrently, `Full text queries` on unstructured data, and `analyzed`
    - often long texts
    - don't what values it contains, match values that `include` a term
    - e.g. search a word contained in a paragraph
    - full text queries are `analyzed the same as the field` they are querying
        - don't use it with `keyword` fields because `not analyzed`

#### `5.6.1` The `match` query
- matches documents contained one or more terms
- search item is analyzed and look up in the field's inverted index
- 

    GET /products/_search
    {
        "query": {
            "match": {
                "name": "pasta"
            }
        }
    }

- In the above example, "pasta" can appear anywhere in the "name" text.
    - with analyzer, "PASTA" can give back the same result

- by default, the following will query as `OR`, which returns results with either word "pasta" or "chicken"
-
    
    GET /products/_search
    {
        "query": {
            "match": {
                "name": "pasta chicken"
            }
        }
    }

- if `AND` is needed, explicit operator setting
-
    
    GET /products/_search
    {
        "query": {
            "match": {
                "name": "pasta chicken",
                "operator": "AND"
            }
        }
    }

#### `5.6.2` Relevance Scoring

- metafield `_score = 1.0` for `terms level query`, or not match
- differently, for `full text queries`, `_score` higher for "better" match
    - Elasticsearch calculates the relevance score for all matches and sorted

#### `5.6.3` Search Multiple fields

- Searching both fields
-
    
    GET /products/_search
    {
        "query": {
            "multi_match": {
                "query": "vegetable",
                "fields": ["name", "tags"]
            }
        }
    }

- `Boost fields`, make some results stand out based on `relevance score`, in the following example, match on "name" will be significant
-
    
    GET /products/_search
    {
        "query": {
            "multi_match": {
                "query": "vegetable",
                "fields": ["name^2", "tags"]
            }
        }
    }

- One field is used for relevance scoring by default as tie breaker
-
    
    GET /products/_search
    {
        "query": {
            "multi_match": {
                "query": "vegetable",
                "fields": ["name", "tags", "description", "ingredients"],
                "tie_breaker": 0.3
            }
        }
    }