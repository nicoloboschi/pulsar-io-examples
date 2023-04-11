# Elastic search sink


## Setup Pulsar and ES cluster
```
docker-compose up -d
```

## Install ElasticSearch Sink

```
docker exec -it pulsar bin/pulsar-admin sinks create --namespace default --tenant public \
    --sink-type elastic_search \
    --inputs es-topic \
    --name elastic-sink \
    --sink-config '{"elasticSearchUrl": "http://elasticsearch:9200", "indexName": "myindex-001", "schemaEnable": true, "keyIgnore": false, "createIndexIfNeeded": true}' \
    --subs-position Earliest \
    --parallelism 1
```

Note: if you get `Function worker service is not done initializing. Please try again in a little while.`, just wait a couple of seconds and retry.


Check sink logs:
```
docker exec -it pulsar cat logs/functions/public/default/elastic-sink/elastic-sink-0.log
```


## Produce messages with schema

```
echo "{\"value\":\"myvalue\",\"inner\":{\"innervalue\":124}}" > /tmp/messagecdclike.json
docker cp /tmp/messagecdclike.json pulsar:/pulsar/message.json

docker exec -it pulsar bin/pulsar-client produce \
    -k "{\"key\":7}" \
    -f message.json \
    -kvet separated \
    -ks "json:{\"type\": \"record\",\"namespace\": \"com.example\",\"name\": \"KeySchema\", \"fields\": [{ \"name\": \"key\", \"type\": \"int\" }]}" \
    -vs "json:{\"type\": \"record\",\"namespace\": \"com.example\",\"name\": \"VSchema\", \"fields\": [{ \"name\": \"value\", \"type\": \"string\" }, { \"name\": \"inner\", \"type\": {\"type\":\"record\",\"namespace\": \"com.example\",\"name\": \"VSchemaInner\", \"fields\":[{\"name\":\"innervalue\", \"type\":\"int\"}] }}]}" \
    es-topic
```


## Check Elastic index 

```
curl "http://localhost:9200/myindex-001/_count?pretty"
curl "http://localhost:9200/myindex-001/_search?pretty"
```

## Check on Kibana

```
open http://localhost:5601
```




