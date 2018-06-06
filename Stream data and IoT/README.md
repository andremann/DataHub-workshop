# Session: Stream data and IoT

Slides are available here: https://drive.google.com/open?id=1oSj0FM02GdWm3SC-XSXv-Cx7A06buMqZgWkCxlP9yik

## Key points
1. Get sense of out data
1. Lightweight prototyping of dashboards
1. Get started with Elasticsearch and Kibana

## Task 1: Dissecting bank accounts
### Requirements for this task
- Dataset: https://download.elastic.co/demos/kibana/gettingstarted/accounts.zip

### What to do
1. Load the dataset on Elasticsearch `curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json`
1. Head to Kibana  -> http://212.219.130.96:5601
1. Create the index pattern on Kibana
1. Slice and dice the dataset

## Task 2: Twitter Streaming API
### Requirements for this task
- Twitter API keys

### What to do
1. Launch the provided Python script and collect some
2. Go on Kibana and have fun

