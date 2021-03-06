# Table

A **table** is a logical abstraction to refer to a collection of related data. It consists of columns and rows \(documents\). 

Data in Pinot tables is sharded into [segments](segment.md). A Pinot table is modeled as a Helix resource. Each segment of a table is modeled as a Helix Partition.

A table is typically associated with a [schema](../../configuration-reference/schema.md), which is used to define the names, data types and other information of the columns of the table.

There are 3 types of a Pinot table 

| Table type | Description |
| :--- | :--- |
| **Offline** | Offline tables ingest pre-built pinot-segments from external data stores. |
| **Realtime** | Realtime tables ingest data from streams \(such as Kafka\) and build segments. |
| **Hybrid** | A hybrid Pinot table has both realtime as well as offline tables under the hood.  |

Note that the query does not know the existence of offline or realtime tables. It only specifies the table name in the query. For example, regardless of whether we have an offline table `myTable_OFFLINE` , or a realtime table `myTable_REALTIME` or a hybrid table containing both of these, the query will simply use `mytable` as `select count(*) from myTable` .

A **table config file** is used to define the table properties, such as name, type, indexing, routing, retention etc. It is written in JSON format, and stored in the property store in Zookeeper, along with the table schema. 

## 

## Creating a table

Create a table config for your data, or see [`examples`](https://github.com/apache/incubator-pinot/tree/master/pinot-tools/src/main/resources/examples) for all possible batch/streaming tables. 

**Prerequisites**

1. S[etup the cluster](cluster.md#setup-a-pinot-cluster)  
2. [Create broker and server tenants](tenant.md#creating-a-tenant)

### Offline Table Creation

{% tabs %}
{% tab title="Docker" %}
```bash
docker run \
    --network=pinot-demo \
    --name pinot-batch-table-creation \
    ${PINOT_IMAGE} AddTable \
    -schemaFile examples/batch/airlineStats/airlineStats_schema.json \
    -tableConfigFile examples/batch/airlineStats/airlineStats_offline_table_config.json \
    -controllerHost pinot-controller \
    -controllerPort 9000 \
    -exec
```

**Sample Console Output**

```bash
Executing command: AddTable -tableConfigFile examples/batch/airlineStats/airlineStats_offline_table_config.json -schemaFile examples/batch/airlineStats/airlineStats_schema.json -controllerHost pinot-controller -controllerPort 9000 -exec
Sending request: http://pinot-controller:9000/schemas to controller: a413b0013806, version: Unknown
{"status":"Table airlineStats_OFFLINE succesfully added"}
```
{% endtab %}

{% tab title="Using launcher scripts" %}
```bash
bin/pinot-admin.sh AddTable \
    -schemaFile examples/batch/airlineStats/airlineStats_schema.json \
    -tableConfigFile examples/batch/airlineStats/airlineStats_offline_table_config.json \
    -exec
```
{% endtab %}

{% tab title="curl" %}
```bash
# add schema
curl -F schemaName=@airlineStats_schema.json  localhost:9000/schemas

# add table
curl -i -X POST -H 'Content-Type: application/json' \
    -d @airlineStats_offline_table_config.json localhost:9000/tables
```
{% endtab %}
{% endtabs %}

Check out the table config in the [Rest API](http://localhost:9000/help#!/Table/alterTableStateOrListTableConfig) to make sure it was successfully uploaded.

### Streaming Table Creation

{% tabs %}
{% tab title="Docker" %}
**Start Kafka**

```
docker run \
	--network pinot-demo --name=kafka \
	-e KAFKA_ZOOKEEPER_CONNECT=pinot-zookeeper:2181/kafka \
	-e KAFKA_BROKER_ID=0 \
	-e KAFKA_ADVERTISED_HOST_NAME=kafka \
	-d wurstmeister/kafka:latest
```

**Create a Kafka Topic**

```text
docker exec \
  -t kafka \
  /opt/kafka/bin/kafka-topics.sh \
  --zookeeper pinot-zookeeper:2181/kafka \
  --partitions=1 --replication-factor=1 \
  --create --topic flights-realtime
```

**Create a Streaming table**

```
docker run \
    --network=pinot-demo \
    --name pinot-streaming-table-creation \
    ${PINOT_IMAGE} AddTable \
    -schemaFile examples/stream/airlineStats/airlineStats_schema.json \
    -tableConfigFile examples/docker/table-configs/airlineStats_realtime_table_config.json \
    -controllerHost pinot-controller \
    -controllerPort 9000 \
    -exec
```

**Sample output**

```text
Executing command: AddTable -tableConfigFile examples/docker/table-configs/airlineStats_realtime_table_config.json -schemaFile examples/stream/airlineStats/airlineStats_schema.json -controllerHost pinot-controller -controllerPort 9000 -exec
Sending request: http://pinot-controller:9000/schemas to controller: 8fbe601012f3, version: Unknown
{"status":"Table airlineStats_REALTIME succesfully added"}
```
{% endtab %}

{% tab title="Using launcher scripts" %}
**Start Kafka-Zookeeper**

```text
bin/pinot-admin.sh StartZookeeper -zkPort 2191
```

**Start Kafka**

```text
bin/pinot-admin.sh  StartKafka -zkAddress=localhost:2191/kafka -port 19092
```

**Create stream table**

```text
bin/pinot-admin.sh AddTable \
    -schemaFile examples/stream/airlineStats/airlineStats_schema.json \
    -tableConfigFile examples/stream/airlineStats/airlineStats_realtime_table_config.json \
    -exec
```
{% endtab %}
{% endtabs %}

Check out the table config in the [Rest API](http://localhost:9000/help#!/Table/alterTableStateOrListTableConfig) to make sure it was successfully uploaded.

