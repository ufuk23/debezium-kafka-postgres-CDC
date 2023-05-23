# debezium-kafka-postgres-CDC
Debezium kafka example - CDC pattern

![debezium](https://user-images.githubusercontent.com/8513827/236702350-b347c23d-e314-468c-8aeb-98cb233c7de3.png)


<h1>Version.1</h1>

<h3> Postgresql config </h3>

psql -U docker -d exampledb -W

CREATE TABLE student (id integer primary key, name varchar);

select * from student;

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
