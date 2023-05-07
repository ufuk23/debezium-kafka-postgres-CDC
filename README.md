# debezium-kafka-postgres-CDC
Debezium kafka example - CDC pattern


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

<h4>Kafka native sh files</h4>  

/usr/bin# kafka-topics --list --zookeeper zookeeper:2181

/usr/bin# kafka-console-consumer --bootstrap-server kafka:9092  --topic postgres.public.student --from-beginning --timeout-ms 10 | tail -n 1





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
<h3> Run the container to tail kafka messages </h3>
Use kafdrop running as a container on port 9000
