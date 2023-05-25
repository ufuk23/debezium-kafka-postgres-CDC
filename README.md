# debezium-kafka-postgres-CDC
Debezium kafka example - CDC pattern

![debezium](https://user-images.githubusercontent.com/8513827/236702350-b347c23d-e314-468c-8aeb-98cb233c7de3.png)


<h1>Version.1</h1>

<h3> Postgresql config </h3>

```
psql -U docker -d exampledb -W

CREATE TABLE student (id integer primary key, name varchar);

select * from student;

ALTER TABLE student REPLICA IDENTITY FULL;

```
<h3>Alter replica indetity as full to get all change log in the message payload</h3>
ALTER TABLE student REPLICA IDENTITY FULL;


<h3> HTTP post to create a debezium connector </h3>

```
{
  "name": "exampledb-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "plugin.name": "pgoutput",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "docker",
    "database.password": "docker",
    "database.dbname": "exampledb",
    "database.server.name": "postgres",
    "table.include.list": "public.student"
  }
}
```

<h3> Run the container to tail kafka messages </h3>
Use kafdrop running as a container on port 9000


![kafdrop0](https://user-images.githubusercontent.com/8513827/236702488-db78ecd7-0ab5-457a-bb33-89fe3ce9fd05.png)


![kafdrop01](https://user-images.githubusercontent.com/8513827/236702559-7d091b30-c586-400b-8954-f4cbb9c9bb61.png)


<h1>Version.2 (kafka clustered)</h1>

RUN docker-compose up -d -f debezium-docker-compose.yaml

POST http://10.1.37.187:8083/connectors
```
{
"name": "inventory-connector",
"config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "postgres",
    "database.password": "postgres",
    "database.dbname" : "postgres",
    "database.server.name": "dbserver1",
    "table.include.list": "inventory.customers"
  }
}
```
<h3>We can tail kafka messages using conduktor, kafka UI or kafdrop containers.</h3>

<h3>Conduktor kafka UI supplies many features managing topics, partitions, replications, consumer groups, connectors, message contents etc.</h3>

![conduktor1](https://github.com/ufuk23/debezium-kafka-postgres-CDC/assets/8513827/0190cd57-66b9-4e9d-b108-07b0f72315f5)

<h3>It allows to produce random message data as well</h3>

![conduktor2](https://github.com/ufuk23/debezium-kafka-postgres-CDC/assets/8513827/9931970e-ce31-4d94-a526-8045434f631a)


<h3>Config Detailed</h3>

```
{
  "name": "pg-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "plugin.name": "pgoutput",
    "database.hostname": "127.0.0.1",
    "database.port": "5432",
    "database.user": "debezium",
    "database.password": "definitelynotpassword",
    "database.dbname" : "dbname",
    "database.server.name": "pg-dev",
    "table.include.list": "public.(.*)",
    "heartbeat.interval.ms": "5000",
    "slot.name": "dbname_debezium",
    "publication.name": "dbname_publication",
    "transforms": "AddPrefix",
    "transforms.AddPrefix.type": "org.apache.kafka.connect.transforms.RegexRouter",
    "transforms.AddPrefix.regex": "pg-dev.public.(.*)",
    "transforms.AddPrefix.replacement": "data.cdc.dbname"
  }
}

```

With the above settings, the connector operates as follows:

Once started, it connects to the specified database and switches to the initial snapshot mode. In this mode, it sends the initial set of data obtained using a typical SELECT * FROM table_name query to Kafka.
After the initialization is complete, the connector switches to the change data capture mode using PostgreSQL’s WAL files as a source.
A brief description of the properties in use:

* name is the name of the connector that uses the configuration above; this name is later used for interacting with the connector (i.e., viewing the status/restarting/updating the configuration) via the Kafka Connect REST API;
* connector.class is the DBMS connector class to be used by the connector being configured;
* plugin.name is the name of the plugin for the logical decoding of WAL data. The wal2json, decoderbuffs, and pgoutput plugins are available for selection. The first two require the installation of the appropriate DBMS extensions; pgoutput for PostgreSQL (version 10+) does not require additional actions;
* database.* — options for connecting to the database, where database.server.name is the name of the PostgreSQL instance used to generate the topic name in the Kafka cluster;
* table.include.list is the list of tables to track changes in; it has the schema.table_name format and cannot be used together with the table.exclude.list;
* heartbeat.interval.ms — the interval (in milliseconds) at which the connector sends heartbeat messages to a Kafka topic;
* heartbeat.action.query is a query that the connector executes on the source database at each heartbeat message (this option was introduced in version 1.1);
* slot.name is the name of the PostgreSQL logical decoding slot. The server streams events to the Debezium connector using this slot;
* publication.name is the name of the PostgreSQL publication that the connector uses. If it doesn’t exist, Debezium will attempt to create it. The connector will fail with an error if the connector user does not have the necessary privileges (thus, you better create the publication as a superuser before starting the connector for the first time).
* transforms defines the way the name of the target topic is changed:
* transforms.AddPrefix.type specifies that regular expressions are used;
* transforms.AddPrefix.regex specifies the mask used to rename the target topic;
* transforms.AddPrefix.replacement contains the replacing string.


